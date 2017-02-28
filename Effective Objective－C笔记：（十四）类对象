描述Objective－C对象所用的数据结构定义在运行期程序库的头文件中，id类型本身也定义在这里：

    typedef struct objc_object {
        Class isa;
    } *id;
    
由此可见，每个对象结构体的首个成员是Class类的变量。该变量定义了对象所属的类，通常称为“is a”指针。Class类也定义在运行期程序库的头文件中：
    
    typedef struct objc_class *Class;

    struct objc_class {
        Class isa;
        Class super_class;
        const char *name;
        long version;
        long info;
        long instance_size;
        struct objc_ivar_list *ivars;
        struct objc_method_list **methodLists;
        struct objc_cache *cache;
        struct objc_protocol_list *protocols;
    }


此结构体中存放类的元数据，例如类的实例实现了几个方法，具备多少个实例变量等信息。此结构体的首个变量也是isa指针，这说明Class本身也是个OC对象。super_class定义了本类的超类。isa指针指向的那个类叫做“元类”，用来表述类对象本身所具备的元数据。

super_class指针确立了继承关系，而isa指针描述了实例所属的类。通过这张布局关系图即可执行“类型信息查询”。我们可以查出对象能否能响应某个选择子，是否遵从某项协议，并且看出它在类继承体系的位置。

##类型信息查询
可以用类型信息查询方法来检视类继承体系。不同的类型信息查询方法之间有一些小区别。

    //能够判断出对象是否是某个特定类的实例
    isMemberOfClass:
    
    //能够判断出对象是否是某类或者其派生类的实例
    isKindOfClass:
    
例如：
    
    NSMutableDictionary *dic = [NSMutableDictionary new];
    [dict isMemberOfClass:[NSDictionary class]];//NO
    [dict isMemberOfClass:[NSMutableDictionary class]];//YES
    [dict isKindOfClass:[NSDictionary class]];//YES
    [dict isKindOfClass:[NSArray class]];//NO
    

    
    
    