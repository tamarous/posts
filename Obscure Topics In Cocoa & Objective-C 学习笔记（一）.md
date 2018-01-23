#Obscure Topics In Cocoa & Objective-C 学习笔记（一）

这本书介绍了Objective－C和Foundation框架中一些十分重要但又经常被我们忽视的特性。作者是大名鼎鼎的Mattt Thompson，他也是iOS上最流行的网络库AFNetworking及其Swift版本Alamofire的作者。

这篇笔记是对本书的Objective－C部份的记录。

## #pragma
\#pragma命令本来是用来进行编译器优化的，但是现在它更常用的一个用途是对代码进行分块。

    #pragma mark - UITableViewDataSource
    
另外也可以用它来暂时隐藏编译器警告：
    
    #pragma clang diagnostic push    #pragma clang diagnostic ignored "-Wunused-variable"    OSStatus status = SecItemExport(...);    NSCAssert(status == errSecSuccess, @"%d", status);    #pragma clang diagnostic pop
    
    
## nil / Nil / NULL / NSNull
| Symbol | Value | Meaning|
|:------|:-----|:------|
|NULL|(void *)0|literal null value for C pointers|
|nil|(id)0|literal null value for Objective-C objects|
|Nil|(Class)0|literal null value for Objective-C classes|
|NSNull|[NSNull null]|singleton object used to represent null|

nil: 定义某一实例对象为空

Nil: 定义某一类为空

NULL: 在C语言中被用来表示空

NSNull: 集合对象，如NSArray、NSSet和NSDictionary中没法包含nil作为其具体值。因此使用一个特定的对象NSNull来表示nil。NSNull提供了一个类方法：[NSNull null]

    NSMutableDictionary *mutableDictionary = 
    [NSMutableDictionary dictionary];
    
    mutableDictionary[@"someKey"] = [NSNull null];
    
## BOOL / bool / Boolean / NSCFBoolean

BOOL实际上是signed char, 用宏YES和NO来表示真和假，常常被用来表示参数、属性和实例变量。

一个常见的错误代码如下：
    
    static BOOL different(int a,int b) {
        return a - b;
    }

事实上，由于BOOL实际上是signed char，所以结果可能会有些难以预料：

    different(11,10) // YES
    different(10,11) // NO
    different(512,256) // NO
    
不过在64位的iOS系统中，BOOL被定义成了bool类型，从而避免了上述问题。


下面这张表格，是Objective-C中所有的真值类型和数值。

| Name | Type | Header | True | No |
|:------:|:-----:|:------:|:----:|:-----:|
| BOOL | signed char/bool | objc.h | YES | NO |
| bool | _Bool(int) | stdbool.h | TRUE | FALSE |
| Boolean | unsigned char | MacTypes.h | TRUE | FALSE |
| NSNumber | _NSCFBoolean | Foundation.h | @(YES) | @(NO)|


## Equality

NSObject通过isEqual:方法来测试与另外一个对象的相等性。

    - (BOOL) isEqual:(id) object {
        return self == object;
    }
    
如果一个NSObject的子类想要实现自己的isEqual:方法，它需要完成以下几件事：

* 实现**isEqualToClassName:**方法。
* 重载isEqual:来完成类和对象检查。会调用上面的方法。
* 重载hash。     
     
### Hash
类似于四则运算，hash运算也有一些运算律：
* 如果a等于b，那么b等于a。

    [a isEqual:b] => [b isEqual:a]
* 如果a等于b，那么a和b的hash相等。

    [a isEqual:b] => [a hash] == [b hash]
* 但是反过来不能成立。即如果a和b的hash相等，不能推出a等于b。


一个例子：

Person.h
    
    @interface Person
    @property NSString *name;
    @property NSDate: *birtyday;
    
    - (BOOL) isEqualToPerson:(Person *)person;
    @end
    
Person.m

    @implementation Person
    - (BOOL) isEqualToPerson:(Person *)person {
    if (! person) {
        return NO;
    }
    
    BOOL haveEqualNames = (! self.name && !person.name) || [self.name isEqualToString:person.name];
    
    BOOL haveEqualBirthdays = (!self.birthday && ! person.birthday) || [self.birthday isEqualToDate:person.birthday];
    
    return haveEqualNames && haveEqualBirthdays;

    }
    
    - (BOOL) isEqual:(id)object {
    if (self == object ) {
        return YES;
    }
    
    if (! [object isKindOfClass:[Person class]]) {
        return NO;
    }
    
    return [self isEqualToPerson:(Person *)object];
    
    }
    
    - (NSUInteger) hash {
        return [self.name hash] ^ [self.birthday hash];
    }
    @end
    
##Encodings

###Type Encode
@encode，是一个编译器指令，能够返回一个C字符串，该字符串是给定类型的内部表示。

使用type encoding的目的是让OC运行时更快地进行消息分发。

| Code | Meaning |
| :----| :----|
|c | A char|
|i | An int |
| s |A short |
| l |A long is treated as a 32-bit quantiy on 64-bit programs|
| q| A long long |
| C | An unsigned char |
| I | An unsigned int |
| S| An unsigned short |
| L|An unsigned long |
|Q | An unsigned long long |
|f | A float |
| d | A double |
| B | A C++ bool or a C99_Bool |
| v | A void |
| * | A character string(char *) |
| @| An object |
| #| A class object |
| : | A method selector |
| [array type] | An array |
| {name = type...} | A structure |
| (name = type...) | A union |
| bnum | A bit field of num bits |
| ^type | A pointer to type |
| ? | An unknown type |


    NSLog(@"int : %s",@encode(int));
    NSLog(@"float : %s",@encode(float));
    NSLog(@"float * : %s", @encode(float *));
    NSLog(@"char : %s", @encode(char));
    NSLog(@"char * : %s", @encode(char *));
    NSLog(@"BOOL : %s", @encode(BOOL));
    NSLog(@"void : %s", @encode(BOOL));
    NSLog(@"void * : %s",@encode(void *));
    
    NSLog(@"NSObject * : %s", @encode(NSObject *));
    NSLog(@"NSObject : %s",@encode(NSObject));
    NSLog(@"[NSObject] : %s",@encode(typedef([NSObject class])));    NSLog(@"NSError ** : %s", @encode(typeof(NSError **)));
        int intArray[5] = {1, 2, 3, 4, 5};    NSLog(@"int[] : %s", @encode(typeof(intArray)));
          float floatArray[3] = {0.1f, 0.2f, 0.3f}; 
    NSLog(@"float[] : %s", @encode(typeof(floatArray)));
          typedef struct _struct {        short a;        long long b;        unsigned long long c;    } Struct;    NSLog(@"struct     : %s", @encode(typeof(Struct)));   
    
    
Result:

    int : i
    float : f
    float * : ^f
    char : c
    char * : *
    BOOL : c
    void : v
    void * : ^v
    NSObject *:@
    NSObject :#
    [NSObject] : {NSObject=#}
    NSError ** : ^@
    int[] : [5i]
    float[] : [3f]
    struct : {_struct=sqQ}

### Method Encode

| Code | Meaning |
| :---:|:---:|
| r| const |
| n| in |
| N| inout |
| o | out |
| O|bycopy|
| R|byref|
| V|oneway|
 
## C Storage Classes

###auto
auto是默认的变量存储类型。它们的生命期只存在于被定义的区块中。

###register
register并不常用。它和auto十分类似，不过auto变量被分配在栈上，而register则被分配在寄存器中。由于寄存器资源是很宝贵的，所以不当地使用register不仅不能带来运行速度的加快，反而会引来很多问题。

###static
static类型的变量具有如下特性：
1. 存在于方法或函数中的static变量在函数调用之间能保留原来的值。
2. 一个全局的static变量能够被它所在的文件中的任意函数和方法使用。

####单例模式
单例模式是设计模式中非常基本和重要的一种设计模式。

    + (instancetype)sharedInstance {
        static id _sharedInstance = nil;
        static dispatch_once_t onceToken;
        dispatch_once(&onceToken,   ^{
        _sharedInstance = [[self alloc] init];
    });
    
        return _sharedInstance;
    }
    
###extern
static使得变量和函数在某个文件中对所有函数全部可见，而extern能使它们在项目的所有文件中可见。

####全局字符串常量
当程序中需要多次使用一个字符串常量时，可以将它声明为一个外部字符串常量。

通常的做法是在公共头文件中声明，然后在实现文件中定义。

AppDelegate.h

    extern NSString *const kAppErrorDomain;
    
AppDelegate.m

    NSString *const kAppErrorDomain = @"com.example.yourapp.error"
    
####公共函数
和全局字符串常量的使用方法一样。

TransactionStateMachine.h

    typedef NS_ENUM(NSUInteger, TranscationState) {
        TransactionOpened,
        TransactionPending,
        TransactionClosed,
    };
    
    extern NSString * NSStringFromTransactionState(TransactionState state);
    
TransactionStateMachine.m

    NSString * NSStringFromTransactionState(TransactionState state)
    {
        switch(state) {
            case TransactionOpened: return @"Opened"
            case TransactionPending: return @"Pending";
            case TransactionClosed: return @"Closed";
            default: return nil;
        }
    }













