##Objective-C 中的属性@property

属性的关键字可以用来控制属性的特性，这些关键字可以被分为三类，分别是存储器控制、原子性和生命周期特性

### 存取器控制
一个属性可以声明为
* `readwrite`：表示该属性同时拥有setter 和getter 方法
* `readonly`：表示该属性是只读的，只拥有getter 方法

为了使得代码更加明确可读，可以自定义访问器的名字，如：
    
    @property (nonatomic, getter = getFunc, setter = setFunc:) BOOL isHidden;
    
那么Objective-C 就会在类中寻找名字分别为`setFunc:`和`getFunc` 的方法来作为属性isHidden的setter 和getter 方法。

### 原子性
* `atomic`（原子的，默认值）：意味着在某一时刻只能有一个进程访问实例变量。它是线程安全的，但是我们一般不使用这个关键字，因为会影响访问属性的效率。不能显式地使用这个关键字，即当一个属性没有被声明成`nonatomic`时，它就是`atomic`的，如果显式地声明，就会报错。
* `nonatomic`（非原子的）：意味着可以被多个进程访问。使用这个关键字时的效率比较高，因此我们一般在声明属性时都把属性设置成`nonatomic`的

### 生命周期特性

生命周期特性包括：`unsafe_unretained`，`assign`，`strong`, `weak`, `copy`五个关键字，用来对属性的内存使用进行管理。

#### assign
`assign`是默认的，也是最简单的：存方法会将传入的值直接赋给属性变量。它一般用于值类型，如`int`，`NSUInteger`，`BOOL`等。
以下面这段声明及定义为例：

    @property (assign) int averageScore;
    
这段代码等同于实现了下列存方法：

    - (void) setAverageScore:(int)d
    {
        _averageScore = d;
    }
    
#### retain
当属性对象是某一个对象的指针时，就需要使用`retain`，表示属性会持有这个对象，也就是说会增加它的引用计数。

#### weak
`weak`，要求不保留传入的对象。如果该对象被释放，那么相应的实例变量会被自动赋为nil。这样做可以避免产生悬空指针。悬空指针指向的是不再存在的对象，向它发送消息通常会导致程序崩溃。相应的存方法会将传入的对象直接赋为实例变量。

`weak`常用来避免循环引用的产生。 

在实际应用中，如果两个类是逻辑上的父子关系，比如说人和车，父亲和孩子，那么应该让“父”类（这里并不是指C++ 中的那种继承关系）拥有对“子”类对象的强引用，让“子”类拥有对“父”类的弱引用。
另外在委托这种设计模式中也常常将delegate 的属性设置为`weak`。

#### strong
`strong`其实是`retain`的ARC版本，要求保留传入的对象，并放弃原有对象。如果原有对象不再有其他拥有方，就会被释放。
关于strong 和weak 的比较，[stackoverflow](http://stackoverflow.com/questions/11013587/differences-between-strong-and-weak-in-objective-c) 上有一个绝妙的解释：
> It may be helpful to think about strong and weak references in terms of balloons.

>A balloon will not fly away as long as at least one person is holding on to a string attached to it. The number of people holding strings is the retain count. When no one is holding on to a string, the ballon will fly away (dealloc). Many people can have strings to that same balloon. You can get/set properties and call methods on the referenced object with both strong and weak references.

>A strong reference is like holding on to a string to that balloon. As long as you are holding on to a string attached to the balloon, it will not fly away.

>A weak reference is like looking at the balloon. You can see it, access it's properties, call it's methods, but you have no string to that balloon. If everyone holding onto the string lets go, the balloon flies away, and you cannot access it anymore.

#### unsafe_unretained
`unsafe_unretained`要求不保留传入的对象。在Cocoa和Cocoa Touch中有一些类是不支持弱引用的，所以不能使用weak属性或者weak局部变量。这些类包括NSTextView, NSFont和NSColorSpace。如果要使用弱引用来指向这些类，就必须使用不安全的引用，即`unsafe_unretained`。它和weak类似，但是当它指向的对象被释放后，它自身并不会被置为nil。所以当你释放了它指向的那个对象，那么它就变成了一个悬挂指针，因此是不安全的。

#### copy
`copy`特性要求拷贝传入的对象，并将新对象赋予实例变量。在拷贝之后，原来传入的值发生任何变化都不会影响到属性对象。以下面这段代码为例：

    @property (copy) NSString* lastName;
    
    - (void) setLastName:(NSString *)d
    {
        lastName= [d copy];
    }
    
有些类会有特定的子类，这些子类是可修改的。这种类型的对象适合使用`copy` 特性。例如NSString，就有NSMutableString 的子类。而向setLastName: 方法传入NSMutableString 对象是有效的：

    NSMutableString *x = [[NSMutableString alloc] initWithString:@"Ono"];
    [myObj setLastName: x];
    
    //因为setLastName:会拷贝传入的对象，所以修改x不会对实例变量产生影响
    [x appendString:@"Lennon"];
    
因为设置了`copy` 特性，所以lastName 的值就是调用[myObj setLastName: x] 时x的值，
即使后面x 的值发生了改变，lastName 的值也不会受到影响。

另外，当属性是一个block 时，也常常使用copy。

    @property (copy) void (^blockProperty)(void);
    
 


