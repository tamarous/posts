###外部变量

Block对象通常会在其内部使用外部创建的其他变量，如基本类型的变量，或者是指向其他对象的指针。这些外部创建的变量叫做外部变量。当执行Block对象时，为了确保其下的外部变量能够始终存在，Block会捕获这些变量。

对基本类型的变量，捕获意味着程序会拷贝变量的值，并保存在Block对象内的局部变量中。对指针类型的变量，Block会使用强引用。这意味着凡是Block对象用到的对象，都会被保留。所以在相应的Block变量被释放前，这些对象一定不会被释放。这样就可能产生强引用环。

###在Block对象中使用self

如果需要写一个使用self的Block对象，那么必须要多做几步工作来避免强引用环。假设有一个类Employee，在这个类中创建了一个Block对象，每次执行的时候就会打印出这个Employee实例：
    
    myBlock = ^{
        NSLog(@"Employee:%@",self);
    };
    
Employee有一个指向Block对象的指针。这个Block对象会捕获self，所以它有一个指向Employee实例的指针。现在就陷入强引用循环了。

为了打破这个强引用循环，可以先在Block对象外声明一个__weak指针，然后将这个指针指向Block对象使用的self；最后在Block对象中使用这个新的指针：

上述代码应该写为：
    
    __weak Employee *weakSelf = self;
    myBlock = ^{
        NSLog(@"Employee:%@", weakSelf);
    };
    
现在这个Block对象对Employee对象是弱引用，所以强引用循环被打破了。然后由于是弱引用，所以self指向的对象在Block执行的时候可能会被释放。

为了避免这种情况的发生，可以在Block对象内部创建一个对self的局部强引用：

    __weak Employee *weakSelf = self;
    myBlock = ^{
        Employee *innerSelf = weakSelf;
        NSLog(@"Employee: %@",innerSelf);
    };
    
通过创建innerSelf强引用，就可以在Block和Employee实例中再次创建一个强引用循环。但是，由于innerSelf是存在于Block内部的，所以只有在Block执行的时候它才会执行，在Block结束后它会自动消失。

###在Block对象中无意使用self

如果直接在Block中使用实例变量，那么block会捕获self，而不会捕获实例对象。举个例子：

    __weak Employee* weakSelf = self;
    myBlock = ^{
        Employee* innerSelf = weakSelf;
        NSLog(@"Employee: %@", innerSelf);
        NSLog(@"EmployeeID: %d", _employeeID);
    };
    
编译器实际上是这样解读这段代码的：

    __weak Employee* weakSelf = self;
    myBlock = ^{
        Employee* innerSelf = weakSelf;
        NSLog(@"Employee: %@", innerSelf);
        NSLog(@"EmployeeID: %d", self->_employeeID);
    };
    
这样，self就又被Block对象捕获到了。所以我们刚刚精心设计的weakSelf和innerSelf失去了作用。直接使用存取方法可以解决这个问题！

    __weak Employee* weakSelf = self;
    myBlock = ^{
        Employee* innerSelf = weakSelf;
        NSLog(@"Employee: %@", innerSelf);
        NSLog(@"EmployeeID: %d", innerSelf.employeeID);
    };
    
    
###修改外部变量

在Block对象中，被捕获的变量是常数，程序无法修改变量所保存的值。
如果需要在Block内修改某个外部变量，则可以在声明这个变量时，在前面加上__block关键字：

例如：

    __block int counter = 0;
    void (^counterBlock) = ^{ counter++};

如果没有__block关键字的话，编译器就会报错。




