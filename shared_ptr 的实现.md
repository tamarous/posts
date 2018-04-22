# shared_ptr 的实现

## 智能指针

shared_ptr 是C++11后引入的智能指针。智能指针的出现大大简化了C++程序员进行内存管理的逻辑，同时也避免了低级C++程序员可能写出的各种bug。陈硕老师在《Linux多线程服务器端编程》中总结道：

>C++里可能出现的内存问题大致有这么几个：
>   1.  缓冲区溢出。
>   2.  空悬指针/悬挂指针
>   3.  重复释放
>   4.  内存泄漏
>   5.  不配对的new[]/delete
>   6.  内存碎片
>
>正确使用智能指针能很轻易地解决前面5个问题。

shared_ptr 是智能指针中比较典型的一个。下面我们将从源代码的角度来进行分析，看看它是怎么实现的。源代码可以从[GNU](https://gcc.gnu.org/onlinedocs/libstdc++/libstdc++-api-4.6/a01033_source.html) 处进行下载。

## __shared_ptr 的定义

shared_ptr是智能指针，有别于int *、char *等原生指针，它是用类来实现的。

```
template<typename _Tp>
class shared_ptr : public __shared_ptr<_Tp> {
  ...
};
```

从类的定义上来看，`shared_ptr` 继承自 `__shared_ptr`。由类的命名规则，我们可以推测出后者是前者的内部实现，实际上也正是如此。`shared_ptr`的所有函数内部都调用了`__shared_ptr`的同名函数，因此在下文中，将不加区分地使用这两个词。类的定义有200来行，直接贴出来不太容易看，因此我们把这个类的实现按照数据成员、构造函数、操作符重载、成员函数这四个部分来进行划分。

首先给出`__shared_ptr`的类定义：

```
template<typename _Tp, _Lock_policy _Lp>
class __shared_ptr {
	...
}
```

`__shared_ptr`是一个模板类，第一个模板参数是一个类型名，而第二个参数则代表一种锁策略，定义在 gnu_cxx的命名空间中。

```
 using __gnu_cxx::_Lock_policy;
```

### 数据成员

```
private:
	_Tp * _M_ptr;
	__shared_count<_Lp> _M_refcount;
```

`__shared_ptr`内部只有两个私有成员变量，一个是原生指针变量`_M_ptr`，一个是用来表示它所指向的对象的引用计数的 `_M_refcount`。 `_M_ptr`的类型与`shared_ptr`指向的类型有关，比如

```
shared_ptr<int> p_int = make_shared<int>(0);
```

那么p_int中的`_M_ptr`就是一个指向int的指针，而

```
shared_ptr<long> p_long = make_shared<long>(300);
```

p_long中的`_M_ptr`就是一个指向long型的指针。

至于 `_M_refcount`所属的类型 `__shared_count`，我们暂时不去管它，现在只要知道 `_M_refcount`表示的是当前这个对象被多少个其他对象引用了，是一个计数器。下文我们再介绍这个类。

### 构造函数

`__shared_ptr` 有13个构造函数。

```
public: 

    // 常量表达式
    constexpr __shared_ptr()
        : _M_ptr(0), _M_refcount() {
    }

    // 显示地使用原生指针来初始化两个私有成员
    template<typename _Tp1>
    explicit __shared_ptr(_Tp1* __p)
        : _M_ptr(__p), _M_refcount(__p) {

        // 要求_Tp1 和_Tp之间可以进行转化
        __glibcxx_function_requires(_ConvertibleConcept<_Tp1*, _Tp*>)
            static_assert(sizeof(_Tp1) > 0, "incomplete type");
        __enable_shared_from_this_helper(_M_refcount, __p, __p);
    }

    // 使用原生指针和自定义delete函数来初始化私有成员
    // 自定义delete函数会影响引用计数的加减规则
    template<typename _Tp1, typename _Deleter>
    __shared_ptr(_Tp1* __p, _Deleter __d)
        : _M_ptr(__p), _M_refcount(__p, __d) {
        __glibcxx_function_requires(_ConvertibleConcept<_Tp1*, _Tp*>)
            // TODO requires _Deleter CopyConstructible and __d(__p) well-formed
            __enable_shared_from_this_helper(_M_refcount, __p, __p);
    }

    // 使用原生指针、自定义delete函数以及自定义分配函数来初始化私有成员
    // 自定义delete函数以及自定义分配函数会影响引用计数的加减规则
    template<typename _Tp1, typename _Deleter, typename _Alloc>
    __shared_ptr(_Tp1* __p, _Deleter __d, _Alloc __a)
        : _M_ptr(__p), _M_refcount(__p, __d, std::move(__a)) {
        __glibcxx_function_requires(_ConvertibleConcept<_Tp1*, _Tp*>)
            // TODO requires _Deleter CopyConstructible and __d(__p) well-formed
            __enable_shared_from_this_helper(_M_refcount, __p, __p);
    }

    // 使用一个空指针来初始化私有成员
    constexpr __shared_ptr(nullptr_t)
        : _M_ptr(0), _M_refcount() {
    }

    // 使用空指针和自定义delete函数来初始化私有成员
    template<typename _Deleter>
    __shared_ptr(nullptr_t __p, _Deleter __d)
        : _M_ptr(0), _M_refcount(__p, __d) {
    }

    // 使用空指针、自定义delete函数以及自定义分配函数来初始化私有成员
    template<typename _Deleter, typename _Alloc>
    __shared_ptr(nullptr_t __p, _Deleter __d, _Alloc __a)
        : _M_ptr(0), _M_refcount(__p, __d, std::move(__a)) {
    }

    // 拷贝构造函数
    template<typename _Tp1>
    __shared_ptr(const __shared_ptr<_Tp1, _Lp>& __r, _Tp* __p)
        : _M_ptr(__p), _M_refcount(__r._M_refcount) {
    }

    //  generated copy constructor, assignment, destructor are fine.
    // 另一种形式的拷贝构造函数
    template<typename _Tp1, typename = typename
        std::enable_if<std::is_convertible<_Tp1*, _Tp*>::value>::type>
        __shared_ptr(const __shared_ptr<_Tp1, _Lp>& __r)
        : _M_ptr(__r._M_ptr), _M_refcount(__r._M_refcount) {
    }

    // 移动构造函数，见《C++ Primer》第五版第473页
    // 不分配内存，而是直接接管r中的资源
    __shared_ptr(__shared_ptr&& __r)
        : _M_ptr(__r._M_ptr), _M_refcount() {
        _M_refcount._M_swap(__r._M_refcount);
        __r._M_ptr = 0;
    }

    // 移动构造函数
    template<typename _Tp1, typename = typename
        std::enable_if<std::is_convertible<_Tp1*, _Tp*>::value>::type>
        __shared_ptr(__shared_ptr<_Tp1, _Lp>&& __r)
        : _M_ptr(__r._M_ptr), _M_refcount() {
        _M_refcount._M_swap(__r._M_refcount);
        __r._M_ptr = 0;
    }

    // 将一个__weak_ptr转换为__shared_ptr
    template<typename _Tp1>
    explicit __shared_ptr(const __weak_ptr<_Tp1, _Lp>& __r)
        : _M_refcount(__r._M_refcount) {
        __glibcxx_function_requires(_ConvertibleConcept<_Tp1*, _Tp*>)

            // It is now safe to copy __r._M_ptr, as
            // _M_refcount(__r._M_refcount) did not throw.
            _M_ptr = __r._M_ptr;
    }

    // If an exception is thrown this constructor has no effect.
    // 将一个unique_ptr转换为__shared_ptr
    template<typename _Tp1, typename _Del>
    __shared_ptr(std::unique_ptr<_Tp1, _Del>&& __r)
        : _M_ptr(__r.get()), _M_refcount() {
        __glibcxx_function_requires(_ConvertibleConcept<_Tp1*, _Tp*>)
            _Tp1* __tmp = __r.get();
        _M_refcount = __shared_count<_Lp>(std::move(__r));
        __enable_shared_from_this_helper(_M_refcount, __tmp, __tmp);
    }
```

可以看出上述构造函数大致分为三类：一类是对`__shared_ptr`的某些行为进行配置的构造函数，一类是拷贝和移动构造函数，一类是将其他指针转换到本类的构造函数。

### 操作符重载

```
public:
    template<typename _Tp1>
    __shared_ptr&
    operator=(const __shared_ptr<_Tp1, _Lp>& __r) {
        _M_ptr = __r._M_ptr;
        _M_refcount = __r._M_refcount; // __shared_count::op= doesn't throw
        return *this;
    }
    __shared_ptr&
    operator=(__shared_ptr&& __r) {
        __shared_ptr(std::move(__r)).swap(*this);
        return *this;
    }

    template<class _Tp1>
    __shared_ptr&
    operator=(__shared_ptr<_Tp1, _Lp>&& __r) {
        __shared_ptr(std::move(__r)).swap(*this);
        return *this;
    }

    template<typename _Tp1, typename _Del>
    __shared_ptr&
    operator=(std::unique_ptr<_Tp1, _Del>&& __r) {

        // 使用std::move表示传参时，希望对_r进行移动构造，而不是拷贝构造
        __shared_ptr(std::move(__r)).swap(*this);
        return *this;
    }    

    typename std::add_lvalue_reference<_Tp>::type
    operator*() const {
        _GLIBCXX_DEBUG_ASSERT(_M_ptr != 0);
        return *_M_ptr;
    }

    _Tp* operator->() const {
        _GLIBCXX_DEBUG_ASSERT(_M_ptr != 0);
        return _M_ptr;
    }

    explicit operator bool() const // never throws
    {
        return _M_ptr == 0 ? false : true;
    }
```

`__shared_ptr` 重载了=操作符，对它有四种不同的实现。第一种是普通的拷贝赋值，第二和第三种都是  `shared_ptr`的移动赋值，第四种是对`unique_ptr`类型的移动赋值。后三者都使用了`std::move`和`swap` 函数。这两个函数都是为了避免拷贝而设计的：`std::move`表示在拷贝时使用移动拷贝，`swap`则是直接交换两个指针的位置。

对*和->的重载则是为了让`_shared_ptr`也能像原生指针一样有解引用和箭头操作。

### 成员函数

```
public:
	// 使用无参的默认构造函数构建出一个临时对象，然后将它和*this的指针进行交换
	// _M_ptr为空，引用计数变为0
    void reset() {
        __shared_ptr().swap(*this);
    }
	
	// 同上
    template<typename _Tp1>
    void reset(_Tp1* __p) {
        // Catch self-reset errors.
        _GLIBCXX_DEBUG_ASSERT(__p == 0 || __p != _M_ptr);
        __shared_ptr(__p).swap(*this);
    }

	// 同上
    template<typename _Tp1, typename _Deleter>
    void reset(_Tp1* __p, _Deleter __d) {
        __shared_ptr(__p, __d).swap(*this);
    }

	// 同上
    template<typename _Tp1, typename _Deleter, typename _Alloc>
    void reset(_Tp1* __p, _Deleter __d, _Alloc __a) {
        __shared_ptr(__p, __d, std::move(__a)).swap(*this);
    }

    template<typename _Tp1>
    bool owner_before(__shared_ptr<_Tp1, _Lp> const& __rhs) const {
        return _M_refcount._M_less(__rhs._M_refcount);
    }

    template<typename _Tp1>
    bool owner_before(__weak_ptr<_Tp1, _Lp> const& __rhs) const {
        return _M_refcount._M_less(__rhs._M_refcount);
    }
    
    _Tp* get() const {
    	return _M_ptr;
	}

    bool unique() const {
        return _M_refcount._M_unique();
    }

    long use_count() const {
        return _M_refcount._M_get_use_count();
    }

    void swap(__shared_ptr<_Tp, _Lp>& __other) {
        std::swap(_M_ptr, __other._M_ptr);
        _M_refcount._M_swap(__other._M_refcount);
    }

```

reset函数的作用是重置`shared_ptr`，让它指向NULL，引用计数也变回到0。

`owner_before`函数的作用则是：当该`shared_ptr`和`__rhs`的类型同属一个继承层次时，不管他们类型是否相同，他们两都被决议为“相等”。当他们的类型不属于同一继承层次时，比较的为他们所管理指针的地址值的大小。为什么要提供这样的函数呢？因为一个智能指针有可能指向了另一个智能指针指向对象中的某一部分，但又要保证这两个智能指针销毁时，只对那个被指的对象完整地析构一次，而不是两个指针分别析构一次。在这种情况下，指针就可以分为两种，一种是 stored pointer 它是指针本身的类型所表示的对象（可能是一个大对象中的一部分）；另一种是 owned pointer 指向内存中的实际完整对象（这一个对象可能被许多智能指针指向了它里面的不同部分，但最终只析构一次）。owner-based order 就是指后一种情况，如果内存中只有一个对象，然后被许多 shared pointer 指向了其中不同的部分，那么这些指针本身的地址肯定是不同的，也就是operator<()可以比较它们，并且它们都不是对象的 owner，它们销毁时不会析构对象。但它们都指向了一个对象，在owner-based order 意义下它们是相等的。

get()的设计则是考虑到兼容性，在使用指针传参时，有的函数只能接受原生指针，而不能接受智能指针，这是我们就可以用get()来获得一个原生指针。

unique()函数返回一个bool型，表示当前这个share_ptr是不是唯一一个管理着所指对象的指针。

use_count()函数返回当前所指对象的引用计数。

##  __shared_count的实现

正如这个类的名字所暗示的那样，`__shared_count` 是一个用于计数的类。`shared_ptr`之所以可以实现对所指向对象的自动管理，与这个计数器类有着非常紧密的联系。

```
template<_Lock_policy _Lp>
class __shared_count {
	...
};
```

### 数据成员

```
private:
	_Sp_counted_base<_Lp>*  _M_pi;
```

`__shared_count`类只有一个成员变量 `_M_pi`，它是一个指向 `_Sp_counted_base`类型的指针。 `_Sp_counted_base` 是一个抽象基类，所以在构造函数里， `_M_pi`会被初始化为一个指向 `_Sp_counted_base`子类的指针 。在通常情况下，它都会初始化为一个`__Sp_counted_ptr`类型的指针。该类是`_Sp_counted_base`类的子类。

### 构造函数

构造器在这里不再详细列出，只需要知道构造器里完成对 `_M_pi`的初始化即可。

### 操作符重载


```
__shared_count& operator=(const __shared_count& __r) {
    _Sp_counted_base<_Lp>* __tmp = __r._M_pi;
    if(__tmp != _M_pi) {
    	if(__tmp != 0)
    	__tmp->_M_add_ref_copy();
    	if(_M_pi != 0)
    		_M_pi->_M_release();
    	_M_pi = __tmp;
    }
    return *this;
}
```

`_shared_count` 只重载了=这个操作符。在这个函数里，首先排除自赋值的情况，然后获得参数__r 的 `_M_pi`指针，用_tmp 来指向它，最后调用 __tmp的 `_M_add_ref_copy`方法，并调用 `_M_pi`的 `_M_release`方法。

### 成员函数


```
// 交换两个_shared_count的_M_pi成员的指针
void _M_swap(__shared_count& __r) {
    _Sp_counted_base<_Lp>* __tmp = __r._M_pi;
    __r._M_pi = _M_pi;
    _M_pi = __tmp;
}

long _M_get_use_count() const {
    return _M_pi != 0 ? _M_pi->_M_get_use_count() : 0;
}

bool _M_unique() const {
    return this->_M_get_use_count() == 1;
}

void* _M_get_deleter(const std::type_info& __ti) const {
    return _M_pi ? _M_pi->_M_get_deleter(__ti) : 0;
}

bool _M_less(const __shared_count& __rhs) const {
    return std::less<_Sp_counted_base<_Lp>*>()(this->_M_pi, __rhs._M_pi);
}

bool _M_less(const __weak_count<_Lp>& __rhs) const {
    return std::less<_Sp_counted_base<_Lp>*>()(this->_M_pi, __rhs._M_pi);
}
```

从重载的=操作符和成员函数中可以看出， `__shared_count`内的函数内部都调用了 `_M_pi`的方法，因此我们转而去看一下这些调用是怎么回事。

##  _Sp_counted_base 的实现

### 成员变量

```
private:
	_Atomic_word  _M_use_count;     // #shared
    _Atomic_word  _M_weak_count;    // #weak + (#shared != 0)
```

成员变量的类型是 `_Atomic_word`，这个类型定义在 atomic_word.h中，实际上就是 int。在 `_Sp_counted_base`初始构造时，将这两个变量的初值都设置为1。

### 构造函数

```
template<_Lock_policy _Lp = __default_lock_policy>
class _Sp_counted_base
    : public _Mutex_base<_Lp> {
    _Sp_counted_base()
        : _M_use_count(1), _M_weak_count(1) {
    }

    virtual ~_Sp_counted_base() {
    }
};
```

### 成员函数

```
	void _M_add_ref_copy() {
		// 原子操作，对_M_use_count加1
        __gnu_cxx::__atomic_add_dispatch(&_M_use_count, 1);
    }

    void _M_release() {
        // Be race-detector-friendly.  For more info see bits/c++config.
        _GLIBCXX_SYNCHRONIZATION_HAPPENS_BEFORE(&_M_use_count);
        if(__gnu_cxx::__exchange_and_add_dispatch(&_M_use_count, -1) == 1) {
            _GLIBCXX_SYNCHRONIZATION_HAPPENS_AFTER(&_M_use_count);
            _M_dispose();
            // There must be a memory barrier between dispose() and destroy()
            // to ensure that the effects of dispose() are observed in the
            // thread that runs destroy().
            // See http://gcc.gnu.org/ml/libstdc++/2005-11/msg00136.html
            if(_Mutex_base<_Lp>::_S_need_barriers) {
                _GLIBCXX_READ_MEM_BARRIER;
                _GLIBCXX_WRITE_MEM_BARRIER;
            }

            // Be race-detector-friendly.  For more info see bits/c++config.
            _GLIBCXX_SYNCHRONIZATION_HAPPENS_BEFORE(&_M_weak_count);
            if(__gnu_cxx::__exchange_and_add_dispatch(&_M_weak_count,
               -1) == 1) {
                _GLIBCXX_SYNCHRONIZATION_HAPPENS_AFTER(&_M_weak_count);
                _M_destroy();
            }
        }
    }

    void _M_weak_add_ref() {
        __gnu_cxx::__atomic_add_dispatch(&_M_weak_count, 1);
    }

    void _M_weak_release() {
        // Be race-detector-friendly. For more info see bits/c++config.
        _GLIBCXX_SYNCHRONIZATION_HAPPENS_BEFORE(&_M_weak_count);
        if(__gnu_cxx::__exchange_and_add_dispatch(&_M_weak_count, -1) == 1) {
            _GLIBCXX_SYNCHRONIZATION_HAPPENS_AFTER(&_M_weak_count);
            if(_Mutex_base<_Lp>::_S_need_barriers) {
                // See _M_release(),
                // destroy() must observe results of dispose()
                _GLIBCXX_READ_MEM_BARRIER;
                _GLIBCXX_WRITE_MEM_BARRIER;
            }
            _M_destroy();
        }
    }

    long _M_get_use_count() const {
        // No memory barrier is used here so there is no synchronization
        // with other threads.
        return const_cast<const volatile _Atomic_word&>(_M_use_count);
    }
```

`上述几个成员函数都是原子性的，保证了线程安全。`

`_M_add_ref_copy`对 `_M_use_count` 进行+1，`_M_weak_add_ref`对`_M_weak_count`进行+1。

`_M_release`中的`__gnu_cxx::__exchange_and_add_dispatch(&_M_use_count, -1)`，先取出 `_M_use_count`的值，然后再对 `_M_use_count`进行-1操作。因此当 `_M_use_count`变为0时，就会调用`_M_dispose()`来释放资源，然后检测`_M_weak_count`的值并进行-1操作，如果也为0，那么调用 `_M_destroy()`来释放本身这个实例对象。

`_M_weak_release` 中所做的事情和`_M_release` 是类似的：先取出 `_M_weak_count`的值，然后对 `_M_weak_count`进行-1操作。当 `_M_weak_count` 变为0时，就会调用  `_M_destroy()`来释放本身这个实例对象。

## __shared_ptr的使用

至此，我们已经把 `__shared_ptr`的内部结构给分析清楚了。不过只介绍这几个内部类，相当于只介绍了人有两个胳膊两条腿，对于人是怎么控制躯体做出各种操作的，还是没有说清楚。因此这一节我们将结合例子来分析一下上述各个类是怎么协调作用的。

### __shared_ptr的拷贝
当我们拷贝一个`shared_ptr` 时，计数器会递增。

#### 拷贝构造

`make_shared`是标准库中的一个函数，它在动态内存中分配一个对象并且初始化该对象，然后返回指向此对象的`shared_ptr`。因此我们可以像下面这样使用它：

```
shared_ptr<int> p = make_shared<int>(42);
```

此时p的引用计数为1。

```
shared_ptr<int> q(p);
```

该过程会导致`__shared_ptr`的拷贝构造函数被调用

```
template<typename _Tp1>
__shared_ptr(const __shared_ptr<_Tp1, _Lp>& __r, _Tp* __p)
: _M_ptr(__p), _M_refcount(__r._M_refcount) {
}
```

p的 `_M_refcount`会用来构造q的 `_M_refcount`，因此会调用 `__shared_count`的拷贝构造函数

```
__shared_count(const __shared_count& __r)
: _M_pi(__r._M_pi) {
    if (_M_pi != 0)
        _M_pi->_M_add_ref_copy();
}
```

在拷贝构造函数中，先进行一次普通的指针赋值，让q 的_M_pi指向p 的`_M_pi`，再调用 q 的`_M_pi`的`_M_add_ref_copy()`方法，让 q 的引用计数+1。

#### 拷贝赋值操作符
当我们用=号来进行拷贝赋值时，会发生什么？还是上面的例子：

```
shared_ptr<int> p = make_shared<int>(42);
shared_ptr<int> q = p;
```
当把 p 赋值给 q 时，会调用`__shared_ptr` 的拷贝赋值函数，因此调用

```
template<typename _Tp1> __shared_ptr& operator=(const __shared_ptr<_Tp1, _Lp>& __r) {
    _M_ptr = __r._M_ptr;
    _M_refcount = __r._M_refcount; // 调用__shared_count 的拷贝赋值运算符
    return *this;
}
```

第一行让q 的`_M_ptr` 指向p 的`_M_ptr`，而第二行对`_M_refcount` 则会调用`__shared_count`的拷贝赋值运算符：

```
__shared_count& operator=(const __shared_count& __r) {
    _Sp_counted_base<_Lp>* __tmp = __r._M_pi;
    if(__tmp != _M_pi) {
    	if(__tmp != 0)
    	__tmp->_M_add_ref_copy();
    	if(_M_pi != 0)
    		_M_pi->_M_release();
    	_M_pi = __tmp;
    }
    return *this;
}
```

拷贝赋值运算符和拷贝构造函数不尽相同。首先获得 p 的`_M_pi`，如果p 和 q 的`_M_pi` 不同，那么就先将 p 指向的对象的引用计数+1，然后调用 q 的_M_pi 的`_M_release`方法。在 《C++ Primer 》中，是这样描述这一过程的：
> 将一个shared_ptr 赋予另一个shared_ptr 会递增赋值号右边 shared_ptr 的引用计数，而递减左侧shared_ptr 的引用计数。

### shared_ptr 自动销毁所管理的对象
`__shared_ptr` 并没有实现析构函数，因此`__shared_ptr` 的析构函数是编译器自动为我们合成的。它会调用每个类成员变量的析构函数，因此在释放`_M_refcount` 时，会调用` __shared_count` 的析构函数：

```
~__shared_count() {
    if (_M_pi != 0)
        _M_pi->_M_release();
}
```

`_M_release`在前文多次出现，因此不再赘述。

## 小结
本文简要介绍了`shared_ptr`的内部实现，分析了其引用计数的原理。C++中的智能指针还有其他几种，如`weak_ptr`、`unique_ptr`等，其特性稍有不同，不过基本思想是类似的。希望本文能对各位有所启发。



































