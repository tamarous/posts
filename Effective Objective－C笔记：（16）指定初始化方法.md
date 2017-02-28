指定初始化方法：为对象提供必要信息以便其能完成工作的初始化方法叫做“指定初始化方法”。

通常情况下，一个类可以有多个初始化方法，以方便他人使用。考虑一个矩形类：

    @interface EOCRectangle:
    @property (nonatomic,assign,readonly) float width;
    @property (nonatomic,assign,readonly) float height;
    @end

为了快速设置这两个属性，这个类可能会提供这样的方法：
    
    －(instancetype) initWithWidth:(float) width andHeight:(float) height
    {
        if ((self = [super init])) {
            _width = width;
            _height = height;
        }
        return self;
    }
    
但是如果有人使用标准的[[EOCRectangle alloc] init]来初始化一个矩形会怎么样呢？这么做是可以的，因为EOCRectangle是继承自NSObject的，所以它会使用NSObject的init方法来初始化全部实例变量，也就是说将全部实例变量设置为0或者与0等价的值。

但是假如我们不想让矩形类的这两个属性为0，因为这样创建出来的矩形可能会带来一些意想不到的结果，所以我们可以指明本类必须用指定初始化方法来初始化。可选的方案是下面两种：

    // 使用默认值
    - (instancetype) init {
        return [self initWithWidth:5.0 andHeight:5.0];
    }
    
    //当调用其他初始化方法时，抛出异常：
    - (instancetype) init {
        @throw [NSException exceptionWithName:NSInternalInconsistencyException
        reason:@"Must use initWithWidth:andHeight: instead."
        userInfo:nil];
    }
    

以上是在使用初始化方法时要注意的一个点。其实在为一个类写初始化方法时，还有很多要注意的地方。简单总结如下：
>1. 在类中定义一个全能初始化方法，并在文档中指明。其他初始化方法均应该调用此方法。

>2. 若全能初始化方法与超类不同，则需要覆写超类中的对应方法。

>3. 如果超类的初始化方法不适用于子类，则应该覆写这个超类方法，并在其中抛出异常。


