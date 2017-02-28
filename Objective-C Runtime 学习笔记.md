# Objective-C Runtime 学习笔记
## Runtime是什么

### 动态 vs 静态语言
Objective-C 是一门面向运行时的语言，也就是说，它把对将要执行什么的推断工作从编译&链接时延后到了运行时。所以你可以根据需要来将消息分发给合适的对象，或者是在运行时交换方法的实现。要实现这些，运行时就必须检查对象，判断对象可以做什么，不可以做什么，然后进行恰当的方法分发。

### 什么是Objectie-C 运行时？

Objective-C 运行时是一个主要用C和汇编语言写成的库，它为C语言添加了面向对象的一些特性，从而形成了Objective—C。运行时要负责诸如加载类信息、进行方法分发、方法传递等一系列工作。

### NSObject 中的方法
Cocoa 中的大部分对象都是NSObject 的子类，继承了NSObject类定义的方法（NSProxy 是一个例外，它不是NSObject 的子类）。因此NSObject 的每个子类实例和类对象都会实现NSObject 的行为。比方说，NSObject 定义了一个名为`description`的方法，这个方法返回一个描述该类内容的字符串，通常是类的名字和对象的首地址。该方法原本被设计用来调试，但是NSObject 的子类可以重写这个方法来返回更多类的细节。

某些方法会向运行时系统查询一些信息，这些方法允许对象进行**自省**。常见的有：

* `class`，用来确认一个对象的类型；
* `isKindOfClass:`和`isMemberOfClass`，用来测试一个对象在继承链中的位置；
* `respondToSelector:`，用来测试一个对象是否能够接受某一消息； 
* `conformsToProtocol:`，用来测试一个对象是否实现了某一个协议的方法；
* `methodForSelector:`，用来确定一个方法实现的地址。 

## 类、对象、方法
在Objective-C 中，类(`objc_class`)、对象(`objc_object`)和方法(`objc_method`)其实都是C的结构体。

### objc_class

    struct objc_class {
        Class isa;
    #if !__OBJC2__
        Class super_class;
        const char *name;
        long version;
        long info;
        long instance_size;
        struct objc_ivar_list *ivars;
        struct objc_method_list **methodLists;
        struct objc_cache *cache;
        struct objc_protocol_list *protocols;
    #endif
    }
    
其中有几个字段是我们所关心的：

* isa: 指向元类（metaclass）的指针。
* super_class：指向该类的父类。如果该类已经是最顶层的根类，则super_class的值为NULL。
* cache：用来缓存最近使用过的方法。
* methodLists：是一个指向`objc_method_list`的指针的指针。从名字上看，应该是用来存储类中的方法的。
* protocols：与methodLists类似，用来存储类所遵循的协议。

### objc_object
objc_object 的定义最为简单：

    struct objc_object {
        Class isa;
    }
可以看出这个结构体中只有一个成员，即为我们上文中提过的isa 指针。

### objc_method 与objc_method_list

    struct objc_method {
        SEL method_name;
        char *method_types;
        IMP method_imp;
    }
这个结构体中有函数名，也就是SEL，有表示函数类型的[字符串](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html#//apple_ref/doc/uid/TP40008048-CH100-SW1)，以及函数的实现IMP。

    struct objc_method_list {
        struct objc_method_list *obsolete;
        int method_count;
    #ifdef __LP64__
        int space;
    #endif
        struct objc_method method_list[1];
    }
从成员上看，`objc_method_list`其实就是一个存储`objc_method`对象的可变长度的数组

## 消息传递机制

### objc_msgSend 函数
在Objective-C 中，消息直到运行时才会被绑定到方法上。编译器会将[receiver selector]这种消息表达式转换为一个`objc_msgSend`调用，而receiver 和selector 被当作参数传入该函数中：

    objc_msgSend(receiver, selector)
如果消息表达式中有更多参数，那么这些参数也都会被传入`objc_msgSend`函数中：

    objc_msgSend(receiver, selector, arg1, arg2,...)
    
消息分发的关键就在于编译器为每个类和对象建立的结构体。每个结构体都包含了两个关键元素：

* 上文提到过的isa 指针。
* 一个class dispatch table。这个表中的每个条目将某个方法的selector 和该方法实现的地址相联系，然后记录在表中。

当一个新的对象被创建时，先分配给这个对象一块内存，然后对它进行初始化。对象的成员变量中有一个指向该类结构的指针，名为`isa`。这个指针给予了对象访问它自身的类及其父类的能力。

![Messaging Framework](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjcRuntimeGuide/Art/messaging1.gif)


当发送一条消息`[receiver selector]`时：
1. 通过receiver的isa 指针找到receiver 的元类；
2. 在receiver 的元类的methodLists 中寻找对应的selector；
3. 如果receiver 的元类中没有selector，那么就继续在receiver 的superclass 中进行寻找；
4. 一旦找到这个selector，就去执行此方法对应的IMP。
用流程图来表示如下：

```flow
st=>start: 开始
e=>end: 结束
op1=>operation: 通过isa 寻找到class
op2=>operation: 寻找selector
op3=>operation: 执行selector 的IMP
con=>condition: 是否找到？

st->op1->op2->con
con(yes)->op3->e
con(no)->op1
```


以上就是运行时方法的动态实现，或者说，在运行时方法被动态绑定到消息上。

为了加速消息分发的过程，运行时系统使用了一个叫做cache的指针来缓存selector 和IMP 。当下一次调用的时候，运行时系统就会首先在cache中进行查找，如果cache 里面没有，才会到methodLists 中查找方法。

### 使用隐藏参数
在上节中，我们提到编译器会将receiver 和selector 以及其他附加参数当作参数传入`objc_msgSend`中，其中receiver 和 selector 是以一种隐藏的方式来进行传递的。虽然没有明确地在方法实现中进行定义，但是它们会在编译时被添加进方法实现中。在方法实现中，也可以引用它们。self 被用来引用receiver，而_cmd 则用来引用selector。

    - strange {
        id target = getTheReceiver();
        SEL method = getTheMethod();
        if (target == self || method == _cmd) {
            return nil;
        } 
        return [target performSelector: method];
    }

### 获得一个方法的地址
唯一能规避动态绑定的方式是获得一个方法的地址然后把它当作一个函数来进行直接调用。当某一方法在一个过程中被大量多次重复调用时，可以使用该方式来避免每次消息分发时的开销。

通过使用一个NSObject 中定义的方法`methodForSelector:`，我们可以获得一个指向方法实现的过程的指针，然后使用这个指针来调用该过程。`methodForSelector:`返回的指针必须被转化为合适的函数类型。

    void (*setter)(id, SEL, BOOL);
    int i;
    
    setter = (void (*)(id, SEL, BOOL))[target methodForSelector:@selector(setFilled:)];
    for(i = 0;i < 1000;i++) {
        setter(targetList[i], @selector(setFilled:), YES);
    }
    
### 动态加载
Objective-C 程序可以在运行时动态加载和链接新的类和范畴。这些新的代码会被合并到程序中，并且和在一开始时就加载的类等同对待。

动态加载能被用来做很多不同的事。系统设置中的不同模块以及Interface Builder 中的自定义调色板都是动态加载的。

## 动态方法解析和转发
如果一个对象不能处理某个消息，那么通常会引发一个`unrecognized selector sent to...`的异常。但是在抛出这个异常之前，运行时系统会给你三次处理该错误的机会。 

* Method resolution
* Fast forwarding
* Normal forwarding

### Dynamic Method Resolution
Objective-C 方法不过是一个接受self 和_cmd 作为参数的C函数。使用函数 `class_addMethod`就可以动态地给某个类添加方法。因此，给出下面这个函数：

    void dynamicMethodIMP(id self, SEL _cmd) {
        // implementation
    }
    
假设MyClass中没有`resolveThisMethodDynamically`这个方法，那么可以像这样使用`resolveInstanceMethod:`或是`resolveClassMethod:`来提供一个函数实现

    @implementation MyClass
    + (BOOL) resolveInstanceMethod:(SEL) aSEL {
        if (aSEL == @selector(resolveThisMethodDynamically)) {
            class_addMethod([self class], aSEL, (IMP)dynamicMethodIMP, "v@:");
            return YES;
        }
        return [super resolveInstanceMethod: aSEL];
    }
    @end
运行时系统会将`dynamicMethodIMP`这个函数添加到MyClass 的方法中。

在iOS 4.3以后，可以使用Block来快速创建一个IMP：

    IMP dynamicIMP = imp_implementationWithBlock:(^(id _self) {
        // implementation
    });
因此上面的程序可以简化为：
    
    class_addMethod([self class], aSEL, dynamicIMP, "v@:");
    
如果resolveXX 方法返回NO，运行时系统就会转向下一步：Fast Forwarding。

### Fast Forwarding
在这个时候，运行时系统试图将这个selector转发给另外一个对象来进行处理：

    -(id) forwardingTargetForSelector:(SEL) aSEL {
        if ([SecondObject respondToSelector: aSEL]) {
            return SecondObject;
        }
        return [super forwardingTargetForSelector: aSEL];
    }

只要这个方法返回的不是nil或者self，整个消息发送的过程就会在SecondObject上重新启动。否则就会继续进行Normal Forwarding。

### Normal Forwarding
当事情真的进行到这一步的时候，就要启用完整的消息转发机制了。
首先运行时系统会发送`methodSignatureForSelector:`消息获得函数的参数和返回值类型，如果返回值为nil，运行时就会发出一个`doesNotRecognizeSelector:`消息，然后程序就挂掉了。如果这返回的是一个函数签名，运行时系统就会创建一个NSInvocation对象，这个对象实际上就是对aSEL的描述，包括aSEL的selector以及各种参数等信息，之后发送`forwardInvocation`消息给receiver，即目标对象。

    - （void) forwardInvocation:(NSInvocation *)anInvocation {
        if ([someOtherObject respondsToSelector:[anInvocation selector]]) {
            [anInvocation invokeWithTarget: someOtherObject];
        } else {
            [super forwardInvocation: anInvocation];
        }
    }


具体来说，当一个对象收到一个不能处理的消息后，运行时系统就会发送给该对象一个`forwardInvocation:`消息，这个消息接收一个NSInvocation对象作为唯一参数。NSInvocation对象会对原来的消息和消息参数进行一次封装。`forwardInvocation:`通常被用来将消息转发到其他对象。

    - （void) forwardInvocation:(NSInvocation *)anInvocation {
        if ([someOtherObject respondsToSelector:[anInvocation selector]]) {
            [anInvocation invokeWithTarget: someOtherObject];
        } else {
            [super forwardInvocation: anInvocation];
        }
    }

## 总结
Objective-C 的运行时系统有着非常强大的功能，以上所记录的内容只是冰山一角。想要对运行时系统有更深入的了解的话，最好的方法是直接去读Objective—C 的[源代码](http://opensource.apple.com/)。


###Reference:
* [Objective-C Runtime](tech.glowing.com/cn/objective-c-runtime/)
* [Objective-C Runtime运行时之一：类与对象](southpeak.github.io/2014/10/25/objective-c-runtime-1/)
* [Objective-C Runtime Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048-CH1-SW1)
* Effective Objective-C，#12 理解消息转发机制



