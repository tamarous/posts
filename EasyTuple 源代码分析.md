# EasyTuple 源代码分析
[`EasyTuple`](https://github.com/meituan/EasyTuple)是由美团开源的一个第三方库，它给Objective-C 添加了元组的能力，可以将几个对象包裹在一个对象中，这样我们就可以从一个函数中返回多个值。它的使用非常简单，比如我们想创建一个由两个元素组成的元组，那么可以这样写：
```
EZTuple2<NSNumber *, NSString *> *tuple = EZTuple(@1, @"string");
```
如果使用 Xcode 辅助编辑器查看预编译后的代码，那么上面的例子在预编译后，会被展开为
```
EZTuple2<NSNumber *, NSString *> *tuple = [[EZTuple2 alloc] initWithFirst:@1 second:@"string"];
```
可以看到原来的宏的写法会自动被转换成 Objective-C 中的类的创建语法了，那么这个转换过程是怎样发生的呢？下面让我们一步步地去分析这个转换的过程。
## EZTuple
右边这个看起来像函数的`EZTuple`，其实是一个宏：
```
#define EZTuple(...) EZTupleAs(EZ_CONCAT(EZTuple, EZ_ARG_COUNT(__VA_ARGS__)), __VA_ARGS__)
```
### EZ_CONCAT
我们遇到的第一个宏就是 `EZ_CONCAT`，它的定义如下
```
#define EZ_CONCAT(A, B) EZ_CONCAT_(A, B)
#define EZ_CONCAT_(A, B) A ## B
```
作用是把 A 和 B 两个字符串连接到一起，比如
`EZ_CONCAT(hello, world)`的结果就是`helloworld`。

### EZ_ARG_COUNT
`EZ_ARG_COUNT`是我们遇到的第二个宏，它的定义有些复杂：
```
#define EZ_ARG_COUNT(...)   _EZ_ARG_COUNT(__VA_ARGS__)
#define _EZ_ARG_COUNT(...)  EZ_ARG_AT(20, ##__VA_ARGS__, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0)

#define EZ_ARG_AT(N, ...)                                    EZ_ARG_AT_(N, __VA_ARGS__)
#define EZ_ARG_AT_(N, ...)                                   EZ_CONCAT(EZ_ARG_AT, N)(__VA_ARGS__)
#define EZ_ARG_AT0(...)                                      EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT1(_0, ...)                                  EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT2(_0, _1, ...)                              EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT3(_0, _1, _2, ...)                          EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT4(_0, _1, _2, _3, ...)                      EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT5(_0, _1, _2, _3, _4, ...)                  EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT6(_0, _1, _2, _3, _4, _5, ...)              EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT7(_0, _1, _2, _3, _4, _5, _6, ...)          EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT8(_0, _1, _2, _3, _4, _5, _6, _7, ...)      EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT9(_0, _1, _2, _3, _4, _5, _6, _7, _8, ...)  EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT10(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, ...)                                                                \
    EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT11(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, ...)                                                           \
    EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT12(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, ...)                                                      \
    EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT13(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, ...)                                                 \
    EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT14(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, ...)                                            \
    EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT15(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, ...)                                       \
    EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT16(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, ...)                                  \
    EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT17(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, ...)                             \
    EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT18(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, ...)                        \
    EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT19(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, ...)                   \
    EZ_ARG_HEAD(__VA_ARGS__)
#define EZ_ARG_AT20(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19, ...)              \
    EZ_ARG_HEAD(__VA_ARGS__)
```

`EZ_ARG_COUNT`是对`_EZ_ARG_COUNT`的一个包装。它会被展开为

```
EZ_ARG_AT(20,##__VA_ARGS__, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0)
```
根据
```
#define EZ_ARG_AT(N, ...)   EZ_ARG_AT_(N, __VA_ARGS__)
#define EZ_ARG_AT_(N, ...)  EZ_CONCAT(EZ_ARG_AT, N)(__VA_ARGS__)
```
上面这个宏就是`EZ_CONCAT(EZ_ARG_AT, 20)(__VA_ARGS__)`，而`EZ_CONCAT(EZ_ARG_AT, 20)`也就是`EZ_ARG_AT20`，因此这个宏进而就等同于
```
EZ_ARG_AT20(##__VA_ARGS__, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0)
```
也就是说我们在使用`EZ_ARG_COUNT(...)`这个宏的时候，它会被最终展开为
```
EZ_ARG_AT20(##__VA_ARGS__, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0)
```
现在假设有这么一行代码`EZ_ARG_COUNT(1,2,3)`，那么它就会展开为
```
EZ_ARG_AT20(1, 2, 3, 20, 19, 18, 17, 16, 15, 14, 13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2, 1, 0)
```
我们注意到
```
#define EZ_ARG_AT20(_0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19, ...)              \
    EZ_ARG_HEAD(__VA_ARGS__)
```
1占据了_0的位置，2占据了_1的位置，3占据了_3的位置，20，19，18，...，4依次占据了
_4、_5、_6、... _19的位置，剩下的3，2，1，0被当做参数传入了 `EZ_ARG_HEAD`中，
因此`EZ_ARG_COUNT(1,2,3)`会被展开为 `EZ_ARG_HEAD(3,2,1,0)`。
### EZ_ARG_HEAD
`EZ_ARG_HEAD`的定义如下
```
#define EZ_ARG_HEAD(FIRST, ...)             FIRST
```
它的作用是取出宏的第一个参数，因此
```
EZ_ARG_HEAD(3,2,1,0) = 3
``` 
也就是
```
EZ_ARG_COUNT(1,2,3) = 3
```
因此`EZ_ARG_COUNT`的作用就是**获得输入的参数的个数**。

由以上三个宏的定义，`EZTuple(@1, @"string")`会被展开为`EZTupleAs(EZTuple2, @1, @"string")`。

进而，根据`EZTupleAs`的定义
```
#define EZTupleAs(_Class_, ...) [[_Class_ alloc] EZ_CONCAT(initWith, EZ_FOR_EACH(EZ_INIT_PARAM_CALL, ,__VA_ARGS__))]
```
`EZTupleAs(EZTuple2, @1, @"string")`会被展开为
```
[[EZTuple2 alloc] EZ_CONCAT(initWith, EZ_FOR_EACH(EZ_INIT_PARAM_CALL, ,@1, @"string"))]
```
### EZ_FOR_EACH
`EZ_FOR_EACH`的定义如下
```
#define EZ_FOR_EACH(...)    _EZ_FOR_EACH(__VA_ARGS__)
#define _EZ_FOR_EACH(MACRO, SEP, ...)   EZ_FOR_EACH_CTX(EZ_FOR_EACH_ITER_, SEP, MACRO, ##__VA_ARGS__)

#define EZ_FOR_EACH_CTX(MACRO, SEP, CTX, ...)                EZ_CONCAT(EZ_FOR_EACH_CTX, EZ_ARG_COUNT(__VA_ARGS__))(MACRO, SEP, CTX, ##__VA_ARGS__)
#define EZ_FOR_EACH_CTX0(MACRO, SEP, CTX)
#define EZ_FOR_EACH_CTX1(MACRO, SEP, CTX, _0) MACRO(0, _0, CTX)
#define EZ_FOR_EACH_CTX2(MACRO, SEP, CTX, _0, _1) \
    EZ_FOR_EACH_CTX1(MACRO, SEP, CTX, _0) SEP MACRO(1, _1, CTX)
#define EZ_FOR_EACH_CTX3(MACRO, SEP, CTX, _0, _1, _2) \
    EZ_FOR_EACH_CTX2(MACRO, SEP, CTX, _0, _1) SEP MACRO(2, _2, CTX)
#define EZ_FOR_EACH_CTX4(MACRO, SEP, CTX, _0, _1, _2, _3) \
    EZ_FOR_EACH_CTX3(MACRO, SEP, CTX, _0, _1, _2) SEP MACRO(3, _3, CTX)
#define EZ_FOR_EACH_CTX5(MACRO, SEP, CTX, _0, _1, _2, _3, _4) \
    EZ_FOR_EACH_CTX4(MACRO, SEP, CTX, _0, _1, _2, _3) SEP MACRO(4, _4, CTX)
// 中间定义都是类似的，为了节省篇幅就不列出了
    EZ_FOR_EACH_CTX18(MACRO, SEP, CTX, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17) SEP MACRO(18, _18, CTX)
#define EZ_FOR_EACH_CTX20(MACRO, SEP, CTX, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18, _19) \
    EZ_FOR_EACH_CTX19(MACRO, SEP, CTX, _0, _1, _2, _3, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15, _16, _17, _18) SEP MACRO(19, _19, CTX)
```

对于`EZTupleAs(EZTuple2, @1, @"string")`展开的结果中的`EZ_FOR_EACH(EZ_INIT_PARAM_CALL, ,@1, @"string")`而言，`MACRO`为`EZ_INIT_PARAM_CALL`，`SEP`为空，`...`为`@1, @"string"`，所以
它会被展开为
```
EZ_FOR_EACH_CTX(EZ_FOR_EACH_ITER_, , EZ_INIT_PARAM_CALL, @1, @"string")
```
根据
```
#define EZ_FOR_EACH_CTX(MACRO, SEP, CTX, ...)   EZ_CONCAT(EZ_FOR_EACH_CTX, EZ_ARG_COUNT(__VA_ARGS__))(MACRO, SEP, CTX, ##__VA_ARGS__)
```
那么对于`EZ_FOR_EACH_CTX(EZ_FOR_EACH_ITER_, , EZ_INIT_PARAM_CALL, @1, @"string")`，`MACRO`为`EZ_FOR_EACH_ITER_`，`SEP`为空，`CTX`为`EZ_INIT_PARAM_CALL`，所以它会被展开为
```
EZ_FOR_EACH_CTX2(EZ_FOR_EACH_ITER_,,EZ_INIT_PARAM_CALL,@1, @"string")
```
再根据
```
#define EZ_FOR_EACH_CTX1(MACRO, SEP, CTX, _0) MACRO(0, _0, CTX)
#define EZ_FOR_EACH_CTX2(MACRO, SEP, CTX, _0, _1) \
    EZ_FOR_EACH_CTX1(MACRO, SEP, CTX, _0) SEP MACRO(1, _1, CTX)
```
`MACRO`为`EZ_FOR_EACH_ITER_`，`SEP`为空，`CTX`为`EZ_INIT_PARAM_CALL`，_0为@1，_1为@"string"，所以上式首先会被展开为
```
EZ_FOR_EACH_CTX1(EZ_FOR_EACH_ITER_, , EZ_INIT_PARAM_CALL, @1) EZ_FOR_EACH_ITER_(1, @"string", EZ_INIT_PARAM_CALL)
```
然后
```
EZ_FOR_EACH_CTX1(EZ_FOR_EACH_ITER_, ,EZ_INIT_PARAM_CALL, @1)
```
中，`MACRO`为`EZ_FOR_EACH_ITER_`，`SEP`为空，`CTX`为`EZ_INIT_PARAM_CALL`，_0为@1，所以它会被展开成
```
EZ_FOR_EACH_ITER_(0,@1,EZ_INIT_PARAM_CALL)
```
所以
```
EZ_FOR_EACH_CTX2(EZ_FOR_EACH_ITER_,,EZ_INIT_PARAM_CALL,@1, @"string")
```
会被展开为
```
EZ_FOR_EACH_ITER_(0, @1, EZ_INIT_PARAM_CALL) EZ_FOR_EACH_ITER_(1, @"string", EZ_INIT_PARAM_CALL)
``` 
也就是说，最一开始的
```
EZ_FOR_EACH(EZ_INIT_PARAM_CALL, ,@1, @"string")
```
被展开为
```
EZ_FOR_EACH_ITER_(0, @1, EZ_INIT_PARAM_CALL) EZ_FOR_EACH_ITER_(1, @"string", EZ_INIT_PARAM_CALL)
``` 
我们先关注`EZ_FOR_EACH_ITER_(0, @1, EZ_INIT_PARAM_CALL)`的展开情况。`EZ_FOR_EACH_ITER_(1, @"string", EZ_INIT_PARAM_CALL)`和它是类似的。
根据
```
#define EZ_FOR_EACH_ITER_(INDEX, PARAM, MACRO)               MACRO(INDEX, PARAM)
#define _EZ_INIT_PARAM_CALL_FIRST(index, param)              EZ_ORDINAL_CAP_AT(index):param
#define _EZ_INIT_PARAM_CALL(index, param)                    EZ_ORDINAL_AT(index):param
#define EZ_INIT_PARAM_CALL(index, param)                     EZ_IF_EQ(0, index)(_EZ_INIT_PARAM_CALL_FIRST(index, param))(_EZ_INIT_PARAM_CALL(index, param))
```
那么`EZ_FOR_EACH_ITER_(0, @1, EZ_INIT_PARAM_CALL)`会被展开为`EZ_INIT_PARAM_CALL(0, @1)`，进而被展开为
```
EZ_IF_EQ(0, 0)(_EZ_INIT_PARAM_CALL_FIRST(0, @1))(_EZ_INIT_PARAM_CALL(0, @1))
```
`EZ_IF_EQ`这个宏的定义如下
```
#define EZ_IF_EQ(A, B)                                       EZ_CONCAT(EZ_IF_EQ, A)(B)

#define EZ_CONSUME_(...)
#define EZ_EXPAND_(...)                                      __VA_ARGS__

#define EZ_IF_EQ0(VALUE)                                     EZ_CONCAT(EZ_IF_EQ0_, VALUE)
#define EZ_IF_EQ0_0(...)                                     __VA_ARGS__ EZ_CONSUME_
#define EZ_IF_EQ0_1(...)                                     EZ_EXPAND_
#define EZ_IF_EQ0_2(...)                                     EZ_EXPAND_
#define EZ_IF_EQ0_3(...)                                     EZ_EXPAND_
#define EZ_IF_EQ0_4(...)                                     EZ_EXPAND_
#define EZ_IF_EQ0_5(...)                                     EZ_EXPAND_
// 中间定义都是类似的，为了节省篇幅就不列出了
#define EZ_IF_EQ0_18(...)                                    EZ_EXPAND_
#define EZ_IF_EQ0_19(...)                                    EZ_EXPAND_
#define EZ_IF_EQ0_20(...)                                    EZ_EXPAND_

#define EZ_IF_EQ1(VALUE)                                        EZ_IF_EQ0(EZ_DEC(VALUE))
#define EZ_IF_EQ2(VALUE)                                     EZ_IF_EQ1(EZ_DEC(VALUE))
#define EZ_IF_EQ3(VALUE)                                     EZ_IF_EQ2(EZ_DEC(VALUE))
#define EZ_IF_EQ4(VALUE)                                     EZ_IF_EQ3(EZ_DEC(VALUE))
#define EZ_IF_EQ5(VALUE)                                     EZ_IF_EQ4(EZ_DEC(VALUE))
// 中间定义都是类似的，为了节省篇幅就不列出了
#define EZ_IF_EQ18(VALUE)                                    EZ_IF_EQ17(EZ_DEC(VALUE))
#define EZ_IF_EQ19(VALUE)                                    EZ_IF_EQ18(EZ_DEC(VALUE))
#define EZ_IF_EQ20(VALUE)                                    EZ_IF_EQ19(EZ_DEC(VALUE))
```

那么`EZ_IF_EQ(0, 0)`会被展开为`EZ_IF_EQ0(0)`，`EZ_IF_EQ0(0)`会被展开为`EZ_IF_EQ0_0`，所以
```
EZ_IF_EQ(0, 0)(_EZ_INIT_PARAM_CALL_FIRST(0, @1))(_EZ_INIT_PARAM_CALL(0, @1))
```
会被展开为
```
EZ_IF_EQ0_0(_EZ_INIT_PARAM_CALL_FIRST(0, @1))(_EZ_INIT_PARAM_CALL(0, @1))
```
`_EZ_INIT_PARAM_CALL_FIRST(0, @1)`会被展开为`EZ_ORDINAL_CAP_AT(0):@1`，`_EZ_INIT_PARAM_CALL(0, @1)`会被展开为`EZ_ORDINAL_AT(0):@1`。
`EZ_ORDINAL_CAP_AT`和`EZ_ORDINAL_AT`的定义如下
```
#define EZ_ORDINAL_AT(N)    EZ_ARG_AT(N, EZ_ORDINAL_NUMBERS)
#define EZ_ORDINAL_CAP_AT(N) EZ_ARG_AT(N, EZ_ORDINAL_CAP_NUMBERS)
#define EZ_ORDINAL_NUMBERS  first, second, third, fourth, fifth, sixth, seventh, eighth, ninth, tenth, eleventh, twelfth, thirteenth, fourteenth, fifteenth, sixteenth, seventeenth, eighteenth, nineteenth, twentieth
#define EZ_ORDINAL_CAP_NUMBERS  First, Second, Third, Fourth, Fifth, Sixth, Seventh, Eighth, Ninth, Tenth, Eleventh, Twelfth, Thirteenth, Fourteenth, Fifteenth, Sixteenth, Seventeenth, Eighteenth, Nineteenth, Twentieth
```
而 `EZ_ARG_AT(N, EZ_ORDINAL_NUMBERS)`这个宏是取`EZ_ORDINAL_NUMBERS`中的第 N 个参数，因此`EZ_ORDINAL_CAP_AT(0):@1`也就是`First:@1`，`EZ_ORDINAL_AT(0):@1`也就是`first:@1`。
所以
```
EZ_IF_EQ0_0(_EZ_INIT_PARAM_CALL_FIRST(0, @1))(_EZ_INIT_PARAM_CALL(0, @1))
```
也就是
```
EZ_IF_EQ0_0(First:@1)(first:@1)
```
由
```
#define EZ_IF_EQ0_0(...)    __VA_ARGS__ EZ_CONSUME_
#define EZ_CONSUME_(...)
```
可知，
```
EZ_IF_EQ0_0(First:@1)(first:@1)
```
会被展开为`First:@1 EZ_CONSUME_(first:@1)`，进而展开为`First:@1`。
同理
```
EZ_FOR_EACH_ITER_(1, @"string", EZ_INIT_PARAM_CALL)
```
展开后得到`Second:@"string"`。
那么，
```
EZ_FOR_EACH(EZ_INIT_PARAM_CALL, ,@1, @"string")
```
最终被展开的结果就是`First:@1 Second:@"string"`，所以
```
EZ_CONCAT(initWith, EZ_FOR_EACH(EZ_INIT_PARAM_CALL, ,@1, @"string"))
```
展开的结果就是`initWithFirst:@1 Second:@"string"`。

因此`EZTupleAs(EZTuple2, @1, @"string")`就是
```
[[EZTuple2 alloc] initWithFirst:@1 Second:@"string"]
```
所以最开始的`EZTuple(@1,@"string")`就被转换为了 Objective-C 中的类的创建语法。

## EZ_TUPLE_CLASSES_DEF
回到最开始的声明：
```
EZTuple2<NSNumber *, NSString *> *tuple = EZTuple(@1, @"string");
```
左边的`EZTuple2`是一个类的名字，但是如果通过Xcode 中的`Go To Definition`来查看这个类的定义的话，会发现 Xcode 将这个类的定义定位到了一个文件中，这个文件里除了头文件外只有一行宏定义：
```
// EZTupleSubClasses.h
#import <Foundation/Foundation.h>
#import <EasyTuple/EZMetaMacros.h>
EZ_TUPLE_CLASSES_DEF
```
`EZ_TUPLE_CLASSES_DEF`这个宏的定义如下
```
#define EZ_TUPLE_CLASSES_DEF    EZ_FOR(20, EZ_TUPLE_DEF_FOREACH, ;)
```
而 `EZ_FOR`的定义则是
```
#define EZ_FOR(COUNT, MARCO, SEP)                            EZ_CONCAT(EZ_FOR, COUNT)(MARCO, SEP)
#define EZ_FOR0(MARCO, SEP)
#define EZ_FOR1(MARCO, SEP)                                  MARCO(0)
#define EZ_FOR2(MARCO, SEP)                                  EZ_FOR1(MARCO, SEP) SEP MARCO(1)
#define EZ_FOR3(MARCO, SEP)                                  EZ_FOR2(MARCO, SEP) SEP MARCO(2)
#define EZ_FOR4(MARCO, SEP)                                  EZ_FOR3(MARCO, SEP) SEP MARCO(3)
#define EZ_FOR5(MARCO, SEP)                                  EZ_FOR4(MARCO, SEP) SEP MARCO(4)
// 中间定义都是类似的，为了节省篇幅就不列出了
#define EZ_FOR19(MARCO, SEP)                                 EZ_FOR18(MARCO, SEP) SEP MARCO(18)
#define EZ_FOR20(MARCO, SEP)                                 EZ_FOR19(MARCO, SEP) SEP MARCO(19)
```
对于`EZ_FOR(20, EZ_TUPLE_DEF_FOREACH, ;)`，`COUNT`为20，`MACRO`为`EZ_TUPLE_DEF_FOREACH`，`SEP`为`;`，因此它会被展开为`EZ_CONCAT(EZ_FOR, COUNT)(MARCO, SEP)`，即`EZ_FOR20(EZ_TUPLE_DEF_FOREACH,;)`。而`EZ_FOR20(EZ_TUPLE_DEF_FOREACH,;)`展开后得到
```
EZ_FOR19(EZ_TUPLE_DEF_FOREACH,;) ; EZ_TUPLE_DEF_FOREACH(19)
```
`EZ_FOR19(EZ_TUPLE_DEF_FOREACH,;)`展开后得到
```
EZ_FOR18(EZ_TUPLE_DEF_FOREACH,;) ; EZ_TUPLE_DEF_FOREACH(18) ; EZ_TUPLE_DEF_FOREACH(18)
```
这样一层层展开，最终的结果为
```
EZ_TUPLE_DEF_FOREACH(0) ; EZ_TUPLE_DEF_FOREACH(1) ; EZ_TUPLE_DEF_FOREACH(2) ; ... EZ_TUPLE_DEF_FOREACH(18); EZ_TUPLE_DEF_FOREACH(19)
```
`EZ_TUPLE_DEF_FOREACH(index)`的定义如下
```
#define EZ_TUPLE_DEF_FOREACH(index) EZ_TUPLE_DEF(EZ_INC(index))
```
内层的`EZ_INC(index)`会对index进行加1操作，那么`EZ_TUPLE_CLASSES_DEF`就会展开为
```
EZ_TUPLE_DEF(1) ; EZ_TUPLE_DEF(2) ; ... EZ_TUPLE_DEF(18) ; EZ_TUPLE_DEF(19) ; EZ_TUPLE_DEF(20)
```
接下来我们再看一下 `EZ_TUPLE_DEF(i)`的定义：
```
#define EZ_TUPLE_DEF(i)                                                                                                         \
@interface EZ_CONCAT(EZTuple, i)<EZ_FOR_COMMA(i, EZ_GENERIC_TYPE)> :EZTupleBase                                                     \
                                                                                                                               \
EZ_FOR_RECURSIVE(i, EZ_PROPERTY_DEF, ;);                                                                                         \
                                                                                                                               \
@property (nonatomic, strong) EZ_CHARS_AT(EZ_DEC(i)) last;                                                                       \
                                                                                                                               \
- (instancetype)EZ_CONCAT(initWith, EZ_FOR_SPACE(i, EZ_INIT_PARAM));                                                              \
                                                                                                                               \
@end
```
可以看出`EZ_TUPLE_DEF(i)`展开后是一个类的定义，并且这个类的定义明显地可以分为三部分：第一部分是拼接出来的类名，该类继承自`EZTupleBase`，第二部分是通过`EZ_FOR_RECURSIVE`生成的属性，第三部分是拼接出来的初始化方法。


### EZ_FOR_COMMA
这个宏出现在类名中，
```
#define EZ_FOR_COMMA(COUNT, MARCO)                           EZ_CONCAT(EZ_FOR_COMMA, COUNT)(MARCO)
#define EZ_FOR_COMMA0(MARCO)
#define EZ_FOR_COMMA1(MARCO)                                 MARCO(0)
#define EZ_FOR_COMMA2(MARCO)                                 EZ_FOR_COMMA1(MARCO), MARCO(1)
#define EZ_FOR_COMMA3(MARCO)                                 EZ_FOR_COMMA2(MARCO), MARCO(2)
#define EZ_FOR_COMMA4(MARCO)                                 EZ_FOR_COMMA3(MARCO), MARCO(3)
#define EZ_FOR_COMMA5(MARCO)                                 EZ_FOR_COMMA4(MARCO), MARCO(4)
// 中间定义都是类似的，为了节省篇幅就不列出了
#define EZ_FOR_COMMA18(MARCO)                                EZ_FOR_COMMA17(MARCO), MARCO(17)
#define EZ_FOR_COMMA19(MARCO)                                EZ_FOR_COMMA18(MARCO), MARCO(18)
#define EZ_FOR_COMMA20(MARCO)                                EZ_FOR_COMMA19(MARCO), MARCO(19)
```
这个宏和上面提过的`EZ_FOR`非常类似，所以展开的结果也是类似的。那么对于`EZ_FOR_COMMA(2, EZ_GENERIC_TYPE)`，展开的结果为
```
EZ_GENERIC_TYPE(0), EZ_GENERIC_TYPE(1)
```

### EZ_GENERIC_TYPE(index)
```
#define EZ_GENERIC_TYPE(index)                               __covariant EZ_CHARS_AT(index): id

#define EZ_CHARS_AT(N)  EZ_ARG_AT(N, EZ_CHARS)

#define EZ_CHARS    A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z
```
`EZ_CHARS_AT(N)`的作用是从`EZ_CHARS`取出第 N 位的字符，因此`EZ_GENERIC_TYPE(0)`会被展开为`__covariant A: id`，`EZ_GENERIC_TYPE(1)`会被展开为`__covariant B: id`，

所以
```
EZ_FOR_COMMA(i, EZ_GENERIC_TYPE)
```
会被展开为
```
__covariant A: id, __covariant B: id
```
`__covariant`这个关键字表示协变性，即子类型可以强转到父类型，是Objective-C 新出现的一个用来表达泛型能力的关键字，与它一同出现的另一个关键字是`____contravariant`，表示逆变性，即父类型可以强转到子类型。对这两个关键字的更详细介绍，可以看一下 sunnyxx 老师的博文[《2015 Objective-C 新特性》](https://blog.sunnyxx.com/2015/06/12/objc-new-features-in-2015/)。

回到类的接口定义，根据上面的分析，
```
@interface EZ_CONCAT(EZTuple, i)<EZ_FOR_COMMA(i, EZ_GENERIC_TYPE)> :EZTupleBase
``` 
会被展开为
```
@interface EZTuple2<__covariant A: id, __covariant B: id)> :EZTupleBase
```
### EZ_FOR_RECURSIVE
这个宏被用于类定义的第二部分，即类的属性的生成中。它的定义和`EZ_FOR`也是类似的：
```
#define EZ_FOR_RECURSIVE(COUNT, MARCO, SEP)                  EZ_CONCAT(EZ_FOR_RECURSIVE, COUNT)(MARCO, SEP)
#define EZ_FOR_RECURSIVE0(MARCO, SEP)
#define EZ_FOR_RECURSIVE1(MARCO, SEP)                        MARCO(0)
#define EZ_FOR_RECURSIVE2(MARCO, SEP)                        EZ_FOR_RECURSIVE1(MARCO, SEP) SEP MARCO(1)
#define EZ_FOR_RECURSIVE3(MARCO, SEP)                        EZ_FOR_RECURSIVE2(MARCO, SEP) SEP MARCO(2)
#define EZ_FOR_RECURSIVE4(MARCO, SEP)                        EZ_FOR_RECURSIVE3(MARCO, SEP) SEP MARCO(3)
#define EZ_FOR_RECURSIVE5(MARCO, SEP)                        EZ_FOR_RECURSIVE4(MARCO, SEP) SEP MARCO(4)
// 中间定义都是类似的，为了节省篇幅就不列出了
#define EZ_FOR_RECURSIVE19(MARCO, SEP)                       EZ_FOR_RECURSIVE18(MARCO, SEP) SEP MARCO(18)
#define EZ_FOR_RECURSIVE20(MARCO, SEP)                       EZ_FOR_RECURSIVE19(MARCO, SEP) SEP MARCO(19)
```
所以
```
EZ_FOR_RECURSIVE(2, EZ_PROPERTY_DEF, ;)
```
就会被展开为
```
EZ_PROPERTY_DEF(0) ; EZ_PROPERTY_DEF(1)
```

### EZ_PROPERTY_DEF(index)
```
#define EZ_PROPERTY_DEF(index)  @property (nonatomic, strong) EZ_CHARS_AT(index) EZ_ORDINAL_AT(index)

#define EZ_ORDINAL_NUMBERS  first, second, third, fourth, fifth, sixth, seventh, eighth, ninth, tenth, eleventh, twelfth, thirteenth, fourteenth, fifteenth, sixteenth, seventeenth, eighteenth, nineteenth, twentieth

#define EZ_ORDINAL_AT(N)    EZ_ARG_AT(N, EZ_ORDINAL_NUMBERS)
```
所以
```
EZ_FOR_RECURSIVE(2, EZ_PROPERTY_DEF, ;)
```
首先被展开为
```
EZ_PROPERTY_DEF(0) ; EZ_PROPERTY_DEF(1)
```
进而被展开为
```
@property (nonatomic, strong) A first;
@property (nonatomic, strong) B second;
```

### EZ_FOR_SPACE
这个宏的名字和`EZ_FOR_COMMA`类似，定义也类似，因此它的作用也是类似的，这里就不浪费篇幅了。`EZ_FOR_SPACE(2, EZ_INIT_PARAM)`会被展开为
```
EZ_INIT_PARAM(0) EZ_INIT_PARAM(1)
```
这里我们也只看一下`EZ_INIT_PARAM(0)`是如何展开的，`EZ_INIT_PARAM(1)`的展开和它是类似的。

```
#define _EZ_INIT_PARAM_FIRST(index)                          EZ_ORDINAL_CAP_AT(index):(EZ_CHARS_AT(index))EZ_ORDINAL_AT(index)
#define _EZ_INIT_PARAM(index)                                EZ_ORDINAL_AT(index):(EZ_CHARS_AT(index))EZ_ORDINAL_AT(index)
#define EZ_INIT_PARAM(index)                                 EZ_IF_EQ(0, index)(_EZ_INIT_PARAM_FIRST(index))(_EZ_INIT_PARAM(index))
```
`EZ_INIT_PARAM(0)`首先被展开为
```
EZ_IF_EQ(0,0)(_EZ_INIT_PARAM_FIRST(0))(_EZ_INIT_PARAM(0))
```
`_EZ_INIT_PARAM_FIRST(0)`会被展开为
```
First:(A)first
```
而`_EZ_INIT_PARAM(0)`会被展开为
```
first:(A)first
```
所以
```
EZ_IF_EQ(0,0)(_EZ_INIT_PARAM_FIRST(0))(_EZ_INIT_PARAM(0))
```
也就是
```
EZ_IF_EQ(0,0)(First:(A)first)(first:(A)first)
```
`EZ_IF_EQ(0,0)`也就是`EZ_IF_EQ0(0)`，也就是`EZ_IF_EQ0_0`，
根据
```
#define EZ_IF_EQ0_0(...)                                     __VA_ARGS__ EZ_CONSUME_
```
`EZ_IF_EQ(0,0)(First:(A)first)(first:(A)first)`就是
```
First:(A)first EZ_CONSUME_(first:(A)first)
```
最终就是`First:(A)first`。即`EZ_INIT_PARAM(0)`最终展开的结果就是`First:(A)first`。同理，`EZ_INIT_PARAM(1)`最终展开的结果就是`Second:(B)second`。
因此，
```
- (instancetype)EZ_CONCAT(initWith, EZ_FOR_SPACE(i, EZ_INIT_PARAM));   
```
展开后的结果就是
```
- (instancetype)initWithFirst:(A)first Second:(B)second;
```
回到一开始的类定义
```
#define EZ_TUPLE_DEF(i)                                                                                                         \
@interface EZ_CONCAT(EZTuple, i)<EZ_FOR_COMMA(i, EZ_GENERIC_TYPE)> :EZTupleBase                                                     \
                                                                                                                               \
EZ_FOR_RECURSIVE(i, EZ_PROPERTY_DEF, ;);                                                                                         \
                                                                                                                               \
@property (nonatomic, strong) EZ_CHARS_AT(EZ_DEC(i)) last;                                                                       \
                                                                                                                               \
- (instancetype)EZ_CONCAT(initWith, EZ_FOR_SPACE(i, EZ_INIT_PARAM));                                                              \
                                                                                                                               \
@end
```
当 i = 2的时候，上面这段宏就会被展开为
```
@interface EZTuple2<__covariant A: id, __covariant B: id)> :EZTupleBase

@property (nonatomic, strong) A first;
@property (nonatomic, strong) B second;
@property (nonatomic, strong) B last;

- (instancetype)initWithFirst:(A)first Second:(B)second;

@end
```
我们在前面分析过，`EZ_TUPLE_CLASSES_DEF`会被展开为
```
EZ_TUPLE_DEF(1) ; EZ_TUPLE_DEF(2) ; ... EZ_TUPLE_DEF(18) ; EZ_TUPLE_DEF(19) ; EZ_TUPLE_DEF(20)
```
所以在预编译时，这些宏就会被展开成`EZTuple1`、`EZTuple2`……`EZTuple20`等类的定义。也就是说，`EasyTuple`通过这个宏为我们一口气定义了20个元祖类。这样的好处是显而易见的：如果不使用宏的话，那么为了创建这么多个类，我们就要手动去书写很多重复的代码；而使用宏的话，则能够通过宏的巧妙组合在预编译的时候自动生成代码。

## 小结
宏是一门非常非常强大的技术，但是之前我一直不知道它有这么多高级的玩法和用法，看了`EasyTuple`的源代码，真是让人大开眼界。其实`EasyTuple`中还有一些上文中没有提到的宏的用法，限于篇幅这里就不一一展开分析了，不过万变不离其宗，只需要像剥洋葱一样一层一层地进行定义的替换，再加一点点耐心和细心，那么再复杂的宏也不在话下。

## 参考资料：
1. [ReactiveCocoa 中奇妙无比的“宏”魔法](https://halfrost.com/reactivecocoa_macro/)
2. [Reactive Cocoa Tutorial [1] = 神奇的Macros](https://blog.sunnyxx.com/2014/03/06/rac_1_macros/)









  

























