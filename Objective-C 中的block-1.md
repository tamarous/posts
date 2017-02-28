# Objective-C 中的block
block 是Apple 为C语言提供的语言扩展，其实质是：带有自动变量的匿名函数。block 在iOS的动画、异步网络请求以及GCD等中被广泛使用。但是，它的语法却有点复杂，有人甚至专门做了一个[网站](http://fuckingblocksyntax.com)来记录block 的用法，从网址中就可以看出他对block 的语法有多少怨言了。

## block 语法

先看一个简单的例子 ：
    
    int multiplier = 7;
    int (^myBlock)(int) = ^(int num) {
        return num * multiplier;
    };
    
myBlock 就是一个block，它由五部分组成：

* `int`是这个block 的返回值类型
* `^`标记myBlock 是一个block，类似于C语言中的指针标记`*`
* `myBlock`是这个block 的名字，类似于函数名
* `int`是参数列表，说明myBlock 接受一个参数且这个参数的类型为int
* `{}`大括号里面的内容是这个block 的表达式

如果不需要参数，那么参数必须指定为void，或者是连同括号一起省略，即参数列表要么是`(void)`，要么就不写。

在使用block 时就如同调用一个函数一样：

    int multiplier = 7;
    int (^myBlock)(int) = ^(int num) {
        return num * multiplier;
    };
    int result = myBlock(8);
    NSLog(@"result is %d",result); // result is 56
    
## block 捕获变量
block 会自动捕捉出现在这个block 之前且和这个block 处于同一个语句块内的变量。

但是要注意的是，对于值类型，block会将这些变量拷贝一份存到block 内，这个过程发生于定义block 的那一瞬间，因此在定义之后，这些变量再怎么变化，block 都看不到了，block 内使用的是它自己留下来的那份拷贝。

举个例子：
    
    int val = 10;
    void (^blk)(void) = ^(void){
        printf("val is %d\n",val);
    };
    val = 20;
    blk();
    
在这个例子中，在定义blk 这个block 时，val 的值为10，因此blk 会将block 内的复制品val 赋值为10。之后block 外面的val 被重新赋了值，但是blk 察觉不到，因此输出结果为
>val is 10

另外，block 中捕获的变量可以认为是`const`的，也就是说block 可以使用，但是不可以对这些变量进行更改。
在上例中，如果试图在block 内对val进行赋值，那么就会出现编译错误。

    int val = 10;
    void (^ blk)(void) = ^(void){
        val = 15; // ERROR!
        printf("val is %d\n",val);
    };
    val = 20;
    blk(); 

如果确实需要在block 内部修改变量的值，那么需要用`__block`来修饰这个需要被修改的变量

    __block int val = 10;
    void (^ blk)(void) = ^(void){
        val = 15; 
        printf("val is %d\n",val);
    };
    val = 20;
    blk();

对于Objective-C 对象，block 截获的是指向该对象底层C语言实现的结构体的指针。
类似于值类型，在不用`__block`修饰的情况下，可以使用被捕获的变量，但是不能对变量进行修改。

    id array = [[NSMutableArray alloc] init];
    id list = [[NSMutableArray alloc] init];
    void (^ blk)(void) = ^{
        id obj = [[NSObject alloc] init];
        [array addObject: obj]; // CORRECT
    };
    
    void (^ blk2)(void) = ^{
        list = [[NSMutableArray alloc] init]; // WRONG
    };
    
在blk2 中，只要在list 的声明前用`__block`修饰，这段代码就可以通过编译。

    __block id list = [[NSMutableArray alloc] init];
    void (^ blk2)(void) = ^{
        list = [[NSMutableArray alloc] init]; // CORRECT
    };


## block 的实现
下面让我们深入进block 的内部，看看block 究竟是怎么实现的。
从clang 的[文档](https://clang.llvm.org/docs/Block-ABI-Apple.html)中我们可以看到用来描述block 的结构体长这个样子：

    struct Block_literal_1 {
        void *isa;
        int flags;
        int reversed; // 实际上是堆上分配的block 的引用计数
        void (*invoke)(void *,...); // 指向block 被编译后的代码的指针
        struct Block_descriptor_1 {
            unsigned long int reserved; // 总是nil
            unsigned long int size; // 整个Block_literal 结构的大小
            // 用来对block进行copy 和dispose 的函数
            void (*copy_helper)(void *dst, void *src);
            void (*dispose_helper)(void *src);
            const char *signature;
        } *descriptor;
        
        // 为每一个block 附近范围内的变量保存一条记录
        // 对于非指针变量，记录的是变量的值，且为const类型
        // 对于指针，则有多种可能，比如__block 指针，对象指针，弱指针，普通指针
    }
    
flags 从这个枚举中取值：
    
    enum {
        BLOCK_HAS_COPY_DISPOSE = (1 << 25),
        BLOCK_HAS_CTOR = (1 << 26),
        BLOCK_IS_GLOBAL = (1 << 28),
        BLOCK_HAS_STRET = (1 << 29),
        BLOCK_HAS_SIGNATURE = (1 << 30),
    };

下面让我们深入进block 的内部，看看block 究竟是怎么实现的。
使用
> clang -rewrite-objc filename

可以将含有block 的源代码转变为C++ 的源代码。

### 最普通的block
我们先对下面这段代码进行转换：

    #include <stdio.h>
    int main() {
        void (^ blk)(void) = ^{
            printf("Hello, World!\n");
        };
        blk();
        return 0;
    }

略去头文件被转换后的部分，我们得到如下代码：

    struct __main_block_impl_0 {
      struct __block_impl impl;
      struct __main_block_desc_0* Desc;
      __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
      }
    };
    static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
    printf("hello world");}
    
    static struct __main_block_desc_0 {
      size_t reserved;
      size_t Block_size;
    } __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
    
    int main()
    {
        void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
        ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
        return 0;
    }
    
可以看到，block 中的语句，现在成为了一个静态函数`__main_block_func_0`，这个函数的参数为`struct __main_block_impl_0 *__cself`，即一个指向`__main_block_impl_0`结构体的指针。让我们把视线转向这个新的结构体：
    
    struct __main_block_impl_0 {
    
        struct __block_impl impl;
        struct __main_block_desc_0* Desc;
        __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
            impl.isa = &_NSConcreteStackBlock;
            impl.Flags = flags;
            impl.FuncPtr = fp;
            Desc = desc;
        }
    };

这个结构体中有两个成员变量和一个构造函数，一个成员是类型为`__block_impl` 的结构体impl，一个成员是类型为指向`__main_block_desc_0`类型结构体的指针Desc。构造函数所做的事就是为这两个成员变量进行赋值。

先看下`__block_impl`结构体：

    struct __block_impl {
        void *isa;
        int Flags;
        int Reserved;
        void *FuncPtr;
    };

* isa指针：指向一个类对象。在非GC模式下有三种类型：`_NSConcreteStackBlock`、`_NSConcreteGlobalBlock`和`_NSConcreteMallocBlock`
* Flags：block 的负载信息，按位存储。
* Reserved：保留变量
* FuncPtr：指向block 函数地址的指针

然后看下第二个结构体：

    struct __main_block_desc_0 {
        unsigned long reserved;
        unsigned long Block_size;
    }
reserved 为今后版本升级所需的区域，Block_size 为一个记录大小的量。

我们再将视线转回到main 函数中，原本定义和使用block 的代码

    void (^ blk)(void) = ^{
            printf("Hello, World!\n");
        };
        blk();
被转化为了
    
    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
    
    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
        
这两行代码很不好看，所以我们先把类型转化的部分都给去掉，得到

    void (*blk)(void) = &__main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA);
    (*blk->impl.FuncPtr)(blk);
    

    
第一行调用了`__main_block_impl_0`的构造函数，blk是一个指向`__main_block_impl_0`类型的指针。因此上述构造函数相当于：

    (blk->impl).isa = &_NSConcreteStackBlock;
    (blk->impl).Flags = 0;
    blk->FuncPtr = __main_block_func_0;
    blk->Desc = &__main_block_desc_0_DATA;

`__main_block_desc_0_DATA`是一个结构体实例：

    static struct __main_block_desc_0 __main_block_desc_0_DATA = {
        0,
        sizeof(struct __main_block_imp1_0)
    };
    
第二行就是使用block 的地方，可以看到，这里通过blk 的成员impl的成员FuncPtr来调用函数，FuncPtr是一个函数指针，它接受的参数是一个指向`__main_block_impl_0`的指针，因此blk 被当作参数传入。

### 捕获自动变量
上面分析的是最最普通情况下的block。接下来我们看下捕获自动变量的block 被转换后的代码与上面的有哪些不同。

    int val = 10;
    void (^ blk)(void) = ^(void){
        printf("val is %d\n",val);
    };

转换后的代码如下：

    struct __main_block_impl_0 {
        struct __block_impl impl;
        struct __main_block_desc_0* Desc;
        int val;
        __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _val, int flags=0) : val(_val) {
            impl.isa = &_NSConcreteStackBlock;
            impl.Flags = flags;
            impl.FuncPtr = fp;
            Desc = desc;
        }
    };

    static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
        int val = __cself->val; // bound by copy

        printf("val is %d\n",val);
    }

    static struct __main_block_desc_0 {
     size_t reserved;
     size_t Block_size;
    } __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
    
    int main()
    {
        int val = 10;
        void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, val));
        ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
        return 0;
    }
    
可以看出，转化后的代码与之前基本相似，但是有两处不同：
1.  结构体`__main_block_impl_0`多了一个成员变量val
2.  结构体`__main_block_impl_0`的构造函数多了一个参数，且这个参数是用来初始化新加入的成员变量的

因此我们可以知道，捕获自动变量值意味着在定义block 的过程中，被捕获的自动变量被保存到block 生成的结构体之中。

### 使用__block 标记的block
前面已经分析了两种block，下面来看下最后一种，即在block 内对变量修改的情况：
源代码：

    __block int val = 10;
    void (* blk)(void) = ^{
        val = 20;
    };
    blk();

转化后：

    struct __Block_byref_val_0 {
    void *__isa;
    __Block_byref_val_0 *__forwarding;
    int __flags;
    int __size;
    int val;
    };

    struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;
    __Block_byref_val_0 *val; // by ref
    __main_block_impl_0(void *fp, struct    __main_block_desc_0 *desc, __Block_byref_val_0 *_val, int flags=0) : val(_val->__forwarding) {
        impl.isa = &_NSConcreteStackBlock;
        impl.Flags = flags;
        impl.FuncPtr = fp;
        Desc = desc;
    }
    };
    
    static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
        __Block_byref_val_0 *val = __cself->val; // bound by ref

        (val->__forwarding->val) = 20;
    }

    static void __main_block_copy_0(struct __main_block_impl_0*dst, struct __main_block_impl_0*src) {
        _Block_object_assign((void*)&dst->val, (void*)src->val, 8/*BLOCK_FIELD_IS_BYREF*/);
    }

    static void __main_block_dispose_0(struct __main_block_impl_0*src) {
        _Block_object_dispose((void*)src->val, 8/*BLOCK_FIELD_IS_BYREF*/);
    }

    static struct __main_block_desc_0 {
        size_t reserved;
        size_t Block_size;
        void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
        void (*dispose)(struct __main_block_impl_0*);
    } __main_block_desc_0_DATA = { 
        0, 
        sizeof(struct __main_block_impl_0), 
        __main_block_copy_0, 
        __main_block_dispose_0
    };

    int main()
    {
        __attribute__((__blocks__(byref))) __Block_byref_val_0 val = {
            (void*)0,
            (__Block_byref_val_0 *)&val, 
            0, 
            sizeof(__Block_byref_val_0), 
            10
        };
        void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_val_0 *)&val, 570425344));
        ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);
        return 0;
    }

让人没想到的是，仅仅是加了一个`__block`修饰符，转换后的代码就多了将尽一倍，仔细一看原来又多了几个新的静态函数和结构体定义，真是让人头大。

没关系，深呼吸，放松一下，我们继续往下看：

这次结构体`__main_block_impl_0`里多出的不是一个单纯的int 类型的变量了，而是一个名为val的指向结构体`__Block_byref_val_0`的指针。

结构体`__Block_byref_val_0`的定义如下：

    struct __Block_byref_val_0 {
        void *__isa;
        __Block_byref_val_0 *__forwarding;
        int __flags;
        int __size;
        int val;
    };


在main 函数中，`__block`变量val 的定义是

    __Block_byref_val_0 val = {
    (void*)0,
    (__Block_byref_val_0 *)&val, 
    0, 
    sizeof(__Block_byref_val_0), 
    10
    };
可以看出，结构体`__Block_byref_val_0`中的`__forwarding`指针指向自身，`__flags`为一个标记位，`__size`表示结构体的大小，而`val`是__block变量的初值。

对`__block`变量val 所做的修改发生在函数指针FuncPtr 所指向的函数`__main_block_func_0`中：

    static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
        __Block_byref_val_0 *val = __cself->val; // bound by ref

        (val->__forwarding->val) = 20;
    }
通过指向自己的`__forwarding`指针访问成员变量`val`，将它的值修改为20。这里有个很奇怪的问题：为什么要大费周章引入`__forwarding`这个指针，而不是直接进行修改呢？

    val->val = 20; // 为什么不这样？
    
这个问题我们留到以后再说。
转化后的代码中还有两个引人注目的静态函数：`__main_block_copy_0`与`__main_block_dispose_0`。

    static void __main_block_copy_0
    (struct __main_block_impl_0*dst, 
    struct __main_block_impl_0*src){
        _Block_object_assign(
            (void*)&dst->val, 
            (void*)src->val, 
            8/*BLOCK_FIELD_IS_BYREF*/);
    }

    static void __main_block_dispose_0
    (struct __main_block_impl_0 *src) {
        _Block_object_dispose(
        (void*)src->val, 
        8/*BLOCK_FIELD_IS_BYREF*/);
    }

当调用[block copy]时，本质上就是调用了
    
    void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
    
而当调用[block release]时，本质上就是调用了

    void (*dispose)(struct __main_block_impl_0*);

`_Block_object_copy`和`_Block_object_dispose`的最后一个参数都是8，这个8代表了`BLOCK_FIELD_IS_BYREF`，定义在头文件Block_private.h 中，用来说明被__block 修饰的是栈区的变量。

## block 的内存管理
### _NSConcreteStackBlock
在结构体`__main_block_impl_0`的构造函数中，有如下一行代码：
    
    impl.isa = &_NSConcreteStackBlock;

这说明该block 的类型为`_NSConcreteStackBlock`，从名字上可以知道，这种block 被分配在栈上。
另外还有两种类型：
* _NSConcreteGlobalBlock，这种block 被分配在程序的数据段(.data)，属于全局的静态block，不会访问任何外部变量
* _NSConcreteMallocBlock，这种block 被分配在由malloc 函数分配的内存中，即堆上，当引用计数为0时会被销毁。

在上面三个例子中，所得到的block 的类型均为`_NSConcreteStackBlock`，那么什么时候才会得到另外两种类型的block 呢？

### _NSConcreteGlobalBlock
我们知道，C++程序中的全局变量会被分配在程序的数据段，因此可以猜想如果block 的定义也发生在全局作用域，那么它的类型可能就成为`_NSConcreteGlobalBlock`了。让我们写代码验证一下：

    void (^ blk)(void) = ^{
        printf("Hello, World!\n");
    };
    
    int main() {
        blk();
        return 0;
    }

转换后：
    
        struct __blk_block_impl_0 {
            struct __block_impl impl;
            struct __blk_block_desc_0* Desc;
            __blk_block_impl_0(void *fp, struct __blk_block_desc_0 *desc, int flags=0)  {
            impl.isa = &_NSConcreteGlobalBlock;
            impl.Flags = flags;
            impl.FuncPtr = fp;
            Desc = desc;
            }
        };
果然！
《Objective-C 高级编程》中介绍说还有另外一种情况下生成的block 也会是`_NSConcreteGlobalBlock`，如下：

    typedef int (^blk)(int);
    for(int rate = 0; rate < 10; rate++) {
        blk blk_t = ^(int count) {
            return count;
        };
    }

这段代码中block 虽然捕获了自动变量rate，但是并没有使用自动变量，因此类型会是`_NSConcreteGlobalBlock`。

注意，在这里，虽然转换后的代码中
    
    impl.isa = &_NSConcreteStackBlock

但是由于clang改写的方式和LLVM不太一样，所以实际上，block 的类型为`_NSConcreteGlobalBlock`，可以在Xcode 中进行验证：
    
    NSLog(@"blk = %@",blk);
输出结果为

    blk = <__NSGlobalBlock__:0x1079340d0>

总结，当是下列两种情况之一时，生成的block 的类型为`_NSConcreteGlobalBlock`
1. block 被定义在全局作用域上
2. block 的表达式中没有使用应该截获的自动变量

### _NSConcreteMallocBlock
`NSConcreteMallocBlock`类型的block 通常不会直接出现，只有在block 被copy的时候，block 的类型才会被修改为`NSConcreteMallocBlock`。
当调用`Block_copy(blk)`或者`[blk copy]`时，会发生block 的copy，也就是会调用
    
    _Block_copy(const void *arg) {
        return _Block_copy_internal(arg, WANTS_ONE);
    }
    
让我们看一下`_Block_copy_internal`的实现

    static void *_Block_copy_internal(const void *arg, const int flags) {
        struct Block_layout *aBlock;
        const bool wantsOne = (WANTS_ONE & flags) == WANTS_ONE;
    
        // 1
        if (!arg) return NULL;
    
        // 2
        aBlock = (struct Block_layout *)arg;
    
        // 3
        if (aBlock->flags & BLOCK_NEEDS_FREE) {
            // latches on high
            latching_incr_int(&aBlock->flags);
            return aBlock;
        }
    
        // 4
        else if (aBlock->flags & BLOCK_IS_GLOBAL) {
            return aBlock;
        }
    
        // 5
        struct Block_layout *result = malloc(aBlock->descriptor->size);
        if (!result) return (void *)0;
    
        // 6
        memmove(result, aBlock, aBlock->descriptor->size); // bitcopy first
    
        // 7
        result->flags &= ~(BLOCK_REFCOUNT_MASK);    // XXX not needed
        result->flags |= BLOCK_NEEDS_FREE | 1;
    
        // 8
        result->isa = _NSConcreteMallocBlock;
    
        // 9
        if (result->flags & BLOCK_HAS_COPY_DISPOSE) {
            (*aBlock->descriptor->copy)(result, aBlock); // do fixup
        }
    
        return result;
    }
这个方法作了下面几件事：
1. 如果传进来的参数为NULL，那么直接返回NULL。
2. 将参数转化为一个指向`Block_layout`结构体的指针aBlock。
3. 如果aBlock 的flag 包含`BLOCK_NEEDS_FREE`，那么这个block 就在堆上，将它的引用计数加1后返回这个block。
4. 如果这个aBlock 是个全局block，那么直接返回它自身。
5. 如果走到了这一步，那么这个aBlock 一定是在栈上分配的了。因此我们使用malloc 在堆上创建一块内存区域result
6. 使用memmove 来将栈上的aBlock 拷贝到刚刚分配的那块内存中去。
7. 设置result的标记位
8. 将result的类型设置为`_NSConcreteMallocBlock`
9. 如果result 拥有一个copy helper函数，那么就会调用这个函数

##小结
本文对block 的相关知识点进行了一些整理，期间查阅了如下的资料，在此向原作者们表示感谢。
###引用
* [How Blocks are implemented(and the consequences)](https://www.cocoawithlove.com/2009/10/how-blocks-are-implemented-and.html)
* [Objective-C 中的Block](https://onevcat.com/2011/11/objc-block/)
* [Objective-C blocks quiz](blog.parse.com/learn/engineering/objective-c-blocks-quiz/)
* [Objective-C中的Block<一>](http://oriochan.com/14710939919047.html)
* [A look inside blocks: Episode 3 (Block_copy)](http://www.galloway.me.uk/2013/05/a-look-inside-blocks-episode-3-block-copy/)
* [谈Objective-C block的实现](http://blog.devtang.com/2013/07/28/a-look-inside-blocks/) 
    

