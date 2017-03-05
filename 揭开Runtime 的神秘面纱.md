# 揭开Runtime 的神秘面纱

## 概述
### 什么是Objectie-C Runtime？
Objective-C Runtime 是一个主要用C和汇编语言写成的库，它为C语言添加了面向对象的一些特性。运行时要负责诸如加载类信息、进行方法分发、方法传递等一系列工作。

### 动态 vs 静态语言
Objective-C 是一门动态语言，刚接触Objective-C 的时候，你一定会为它使用方括号这种怪异的“函数调用”方式而感到惊讶。准确的说，Objective-C 中的如下语句

    [receiver message]
    
并不等同于C语言中的函数调用，而是向receiver 对象发送message 消息。C语言中的函数调用是在编译期间确定的，而Objective-C 是一门面向runtime的语言，也就是说，它把消息发送的时机从编译&链接时延后到了运行时。这正是Objective-C 众多的黑魔法的源头所在。

## Objective-C 中的对象模型
我们知道，C语言是不支持面向对象特性的，而Objective-C 作为一门面向对象语言，却架构在C语言的基础上，这其中Apple 的工程师一定做了相当多的努力。下面就让我们从Objective-C 的[源代码](https://opensource.apple.com/source/objc4/objc4-706/)出发，去看一看Objective-C 是怎么实现这一切的。
在Objective-C 中，类的类型为`Class`。它实质上是一个指向结构体`objc_class`的指针。
    
    typedef struct objc_class *Class;

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

* isa：指向类本身的指针。
* super_class：指向该类的父类。如果该类已经是最顶层的根类，则super_class的值为NULL。
* cache：用来缓存最近使用过的方法。
* methodLists：是一个指向`objc_method_list`的指针的指针，用来存储类中的方法的。
* protocols：与methodLists类似，用来存储类所遵循的协议。

isa 指针指明了当前对象所属的类。实例对象的isa 指向了该实例对象所属的类，然而在Objective-C 中，类本身也是一个对象，称为**类对象**。举例来说：
    
    NSString *str = [NSString stringWithFormat:@"%d",1];
    
在这里，`NSString` 是`stringWithFormat:`消息的接收者，它是一个类对象。
那么有意思的是：类对象的isa 指向谁呢？是指向这个类本身吗？

在Objective-C 中，有一个元类（`metaclass`）的概念，可以解释这个问题。
> The meta-class is the class for a Class object.

类对象的isa 指针指向这个类对象的元类。元类本身也是一个类。当我们向一个实例对象发送消息时，runtime 会在这个对象所属的类的方法列表中寻找方法；当我们向一个类对象发送消息时，runtime 会在这个类对象的元类的方法列表中寻找方法实现。

既然元类本身也是一个类，所以元类也有一个isa 指针。
元类的isa 指向基类（NSObject）的isa。而基类的isa 指针指向它们自身。

在创建类的实例的过程中，会调用`class_createInstance`函数，这个函数的返回值类型是id。id是Objective-C 中表示通用类型的指针，有点类似于C语言中的`void *`。id的定义如下：
    
    typedef struct objc_object *id;

所以id 其实就是objc_object 类型的指针，进而我们推测，Objective-C 中的类的实例，在底层实现上应该都与这个类型有点关系：
    
    struct objc_object {
        Class isa;
    } *id;

可以看出这个结构体中只有一个成员，即为我们上文中提过的isa 指针。事实上，在Objective-C 中，只要某个数据结构中有一个`Class isa`成员，这个数据结构就会被认作是一个`objc_object`。


### objc_method 与objc_method_list

    struct objc_method {
        SEL method_name; // 方法的签名
        char *method_types; // 方法参数的类型
        IMP method_imp; // 方法的指针
    }

关于SEL 和IMP 的关系，可以参考下[这篇文章](www.cocoawithlove.com/2008/02/imp-of-current-method.html)。

    struct objc_method_list {
        struct objc_method_list *obsolete;
        int method_count;
    #ifdef __LP64__
        int space;
    #endif
        struct objc_method method_list[1];
    }
从成员上看，`objc_method_list`其实就是一个存储`objc_method`对象的可变长度的数组。

## 消息传递机制

### 正常的消息分发过程

在Objective-C 中，消息直到运行时才会被绑定到对应的方法上。编译器会将[receiver selector]这种消息表达式转换为一个`objc_msgSend`调用，而receiver 和selector 被当作参数传入该函数中：

    objc_msgSend(receiver, selector)
    

objc_msgSend的函数原型如下：
    
    objc_msgSend(id self, SEL op, ...);
如果消息表达式中有更多参数，那么这些参数也都会被传入`objc_msgSend`函数中：

    objc_msgSend(receiver, selector, arg1, arg2,...)
    

消息分发的关键就在于编译器为每个类和对象建立的结构体。每个结构体都包含了两个关键元素：

* 上文提到过的isa 指针。
* 一个class dispatch table。这个表中的每个条目将某个方法的selector 和该方法实现的地址相联系，然后记录在表中。

当一个新的对象被创建时，先分配给这个对象一块内存，然后对它进行初始化。`isa`指针给予了对象访问它自身的类及其父类的能力。

![Messaging Framework](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjcRuntimeGuide/Art/messaging1.gif)


当发送一条消息`[receiver selector]`时，如果receiver 是一个实例对象：
1. 通过receiver 的isa 指针找到receiver 所属的类；
2. 在receiver 的类的methodLists 中寻找对应的selector；
3. 如果receiver 的类中没有selector，那么就继续在receiver 的superclass 中进行寻找；
4. 一旦找到这个selector，就去执行此方法对应的IMP。

如果receiver 是一个类对象，那么所有的查找都是在类对象的元类中进行的。
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


为了加速消息分发的过程，运行时系统使用了一个叫做cache的指针来缓存selector 和IMP 。当下一次调用的时候，运行时系统就会首先在cache 中进行查找，如果cache 里面没有，才会到methodLists 中查找方法。

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
当某一方法在一个过程中被大量多次重复调用时，我们可以先去获得这个方法的地址然后把它当作一个函数来进行直接调用，这样可以避免每次消息分发时的开销。

通过使用一个NSObject 中定义的方法`methodForSelector:`，我们可以获得一个指向方法实现的指针，然后使用这个指针来调用该过程。`methodForSelector:`返回的指针必须被转化为合适的函数类型。

    void (*setter)(id, SEL, BOOL);
    int i;
    
    setter = (void (*)(id, SEL, BOOL))[target methodForSelector:@selector(setFilled:)];
    for(i = 0;i < 1000;i++) {
        setter(targetList[i], @selector(setFilled:), YES);
    }

## 消息转发机制
如果一个对象不能处理某个消息，那么通常会引发一个`unrecognized selector sent to...`的异常。但是在抛出这个异常之前，运行时系统会给你三次处理该错误的机会。 

* Method resolution
* Fast forwarding
* Normal forwarding

### Dynamic Method Resolution
Objective-C 方法不过是一个接受self 和_cmd 作为参数的C函数。使用函数 `class_addMethod`就可以动态地给某个类添加方法。因此，给出下面这个函数：

    void dynamicMethodIMP(id self, SEL _cmd) {
        // implementation
    }
    
假设MyClass 中没有`resolveThisMethodDynamically`这个方法，那么可以像这样使用`resolveInstanceMethod:`或是`resolveClassMethod:`来提供一个函数实现：

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

在iOS 4.3以后，可以使用block 来快速创建一个IMP：

    IMP dynamicIMP = imp_implementationWithBlock:(^(id _self) {
        // implementation
    });
因此上面的程序可以简化为：
    
    class_addMethod([self class], aSEL, dynamicIMP, "v@:");
    
如果resolveXX 方法返回NO，运行时系统就会转向下一步：Fast Forwarding。

### Fast Forwarding
在这个时候，运行时系统试图将这个selector 转发给另外一个对象来进行处理：

    -(id) forwardingTargetForSelector:(SEL) aSEL {
        if ([SecondObject respondToSelector: aSEL]) {
            return SecondObject;
        }
        return [super forwardingTargetForSelector: aSEL];
    }

只要这个方法返回的不是nil或者self，整个消息发送的过程就会在SecondObject 上重新启动。否则就会继续进行Normal Forwarding。

### Normal Forwarding
当事情真的进行到这一步的时候，就要启用完整的消息转发机制了。
首先运行时系统会发送`methodSignatureForSelector:`消息获得函数的参数和返回值类型，如果返回值为nil，运行时就会发出一个`doesNotRecognizeSelector:`消息，然后程序就挂掉了。如果这返回的是一个函数签名，运行时系统就会创建一个NSInvocation 对象，这个对象实际上就是对aSEL的描述，包括aSEL的selector以及各种参数等信息，之后发送`forwardInvocation`消息给receiver，即目标对象。

    - （void) forwardInvocation:(NSInvocation *)anInvocation {
        if ([someOtherObject respondsToSelector:[anInvocation selector]]) {
            [anInvocation invokeWithTarget: someOtherObject];
        } else {
            [super forwardInvocation: anInvocation];
        }
    }

然后在`someObject`中会从头进行消息查找与分发工作。

## 操纵类的实现
### 与类有关的函数
Runtime 中以class 开头的函数都是用来直接对类进行操作的。我将这些方法按照目的分为了以下几类：用来查询的，用来判断的，用来拷贝的，用来添加的，用于控制类的生命周期的等等。

#### 查询函数
>const char *class_getName(Class cls);

* 获得类的名字。如果传入的cls 为nil，则返回“nil”，否则返回cls->name。

> Class class_getSuperclass(Class cls);

* 获得传入的cls 的超类。
* > size_t class_getInstanceSize(Class cls);

* 获得类中实例对象的大小。

> Ivar class_getInstanceVariable(Class cls, const char *name);

* 获得类中的某个以name 指定的实例成员。

> Ivar class_getClassVariable(Class cls, const char *name);

* 获得类中的某个以name 指定的类成员。

> Method class_getInstanceMethod(Class cls, SEL name);

* 获得类中的某个名为name的实例方法。

> Method class_getClassMethod(Class cls, SEL name);

* 获得类中某个名为name 的类方法。

> IMP class_getMethodImplementation(Class cls, SEL name);

* 获得类中的某个名为name 的方法的具体实现。

> objc_property_t class_getProperty(Class cls, const char *name);

* 获得类中的某个名为name 的属性。



#### 判断函数
> BOOL class_isMetaClass(Class cls);

* 判断传入的cls 是否是一个元类。

> BOOl class_respondsToSelector(Class cls, SEL sel);

* 判断某个类的实例能否响应某个特定的选择子。

> BOOL class_conformsToProtocol(Class cls, Protocol *protocol);

* 判断某个类是否遵循了特定协议。

#### 拷贝函数

> Ivar *class_copyIvarList(Class cls, unsigned int *outCount);

* 返回一个数组，这个数组中的元素是指向类中实例成员变量的指针。一个元素对应一个成员变量。

> Method *class_copyMethodList(Class cls, unsigned int *outCount);

* 返回一个数组，这个数组中的元素是指向类中的实例方法的指针。一个元素对应一个实例方法。

> objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount);

* 返回一个数组，这个数组中的元素是指向类中的属性的指针。一个元素对应一个属性。

> Protocol * __unsafe_unretained *class_copyProtocolList(Class cls, unsigned int *outCount);

* 返回一个数组，这个数组中的元素是指向类所遵循的协议的指针。一个元素对应一个协议。

以上函数中的outCount 的值，是所返回的数组的大小。在使用完这些数组后，必须使用free()来释放掉数组的内存空间。

#### 添加函数
> BOOL class_addMethod(Class cls, SEL name, IMP imp, 
                                 const char *types);
                                 
* 向一个类中添加方法。如果添加成功了就返回YES，否则返回NO。

> IMP class_replaceMethod(Class cls, SEL name, IMP imp, 
                                    const char *types);
                                    
* 更换一个类中的已有方法的实现。name 是要更换的方法的选择子，imp 是新的方法的实现。types 是方法的参数。返回的是被替换的方法的实现。

> BOOL class_addIvar(Class cls, const char *name, size_t size, 
                               uint8_t alignment, const char *types);
                               
* 向一个类中添加某个实例变量。要注意的是，这个方法只能用在`objc_allocateClassPair`和    `objc_registerClassPair`两个函数调用之间。不能用这个方法向一个已经存在的类添加实例变量，因为已经存在的类的内存布局已经固定了。这也是我们**不能在分类中向一个类添加属性**的原因。 

> BOOL class_addProtocol(Class cls, Protocol *protocol);

* 向一个类添加某个协议。

> BOOL class_addProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount);

* 向一个类中添加某个属性。attributes是这个属性应该有的特性，详见[这里](http://www.tamarous.com/2016/05/22/objectivec-zhong-de-shu-xing/)。attributesCount 是特性的数量。

> void class_replaceProperty(Class cls, const char *name, const objc_property_attribute_t *attributes, unsigned int attributeCount);

* 更换某个类中的现有属性。使用方法与上一个方法类似。

#### 类的生命周期函数
> id class_createInstance(Class cls, size_t extraBytes);

* 用于创建类的实例。返回成功创建的这个实例。extraBytes是想在这个类中添加的额外实例变量的大小。

> void *objc_destructInstance(id obj);

* 用于销毁类的实例。CF和其他一些底层类会在垃圾回收环境下调用这个类（此处存疑）。

> Class objc_allocateClassPair(Class superclass, const char *name, 
                                         size_t extraBytes);

* 用来创建一个新的类及其元类。superclass 制定了要创建的这个类的父类。name 是要创建的类的名字。extraBytes 通常设置为0。返回的是新创建的类，或者是nil。
* 在创建了新的类之后，可以通过调用`class_addMethod` 和 `class_addIvar`来为这个新类添加方法及实例变量。在添加完之后，调用`objc_registerClassPair`。这样这个新类就可以使用了。
* 实例方法应该被添加到类中，类方法应该被添加到类的元类中。

> void objc_registerClassPair(Class cls);

* 注册一个刚刚创建的类。只有在这之后，这个类才可以被使用。

### 与Method 有关的函数
与Method 有关的函数很多，但是我们在这只关心一个函数：

> void method_exchangeImplementations(Method m1, Method m2);

可以用这个函数来交换两个方法的实现。Effective Objective-C 在条款13中举了一个例子：用自定义的方法来交换NSString 中的`lowercaseString`实现。假设这个自定义的方法名为`eoc_myLowercaseString`，位于NSString 的一个分类中：
    
    - (NSString *) eoc_myLowercaseString {
        NSString *lowercase =   [self eoc_myLowercaseString];
        NSLog(@"%@ => %@",self, lowercase);
        return lowercase;
    }
     
实现交换的代码如下：

    Method original = class_getInstanceMethod([NSString class], @selector(lowercaseString));
    Method swapped = class_getInstanceMethod([NSString class], @selector(eoc_myLowercaseString));
    
    method_exchangeImplementations(original, swapped);
    
    


## 小结
Objective-C 中的Runtime 的知识点很多，而且不管是在面试还是在实际工作中都有着超高的出场率。因此掌握Runtime 的原理和用途非常有必要。
 
### Reference:
* [Objective-C Runtime运行时之一：类与对象](southpeak.github.io/2014/10/25/objective-c-runtime-1/)
* [Objective-C Runtime](tech.glowing.com/cn/objective-c-runtime/)
* [Objective-C Runtime Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048-CH1-SW1)
* Effective Objective-C，#12 理解消息转发机制
* [What is a meta-class in Objective-C?](https://www.cocoawithlove.com/2010/01/what-is-meta-class-in-objective-c.html)



