#Obscure Topics In Cocoa & Objective-C 学习笔记（二）

## instancetype

alloc和init都返回id类型，但是Xcode却能对它们进行正确的类型检查，这是怎么做到的呢？其实这是由于在Cocoa中，有一个惯例：凡是名字中有alloc或者init的函数或者方法，返回值类型均是这个类型的实例。

类构造器，虽然也返回id类型，但是由于它们不遵循这个惯例，所以并不能得到编译器的类型检查。

使用instancetype，会告诉编译器，某个方法的返回值就是此方法所在类的类型。例如：

    @interface Person
    + (instancetype) personWithName:(NSString *)name;
    @end
    
由于使用了instancetype，编辑器会正确推断出personWithName的结果是Person类型的实例。

## NS\_ENUM & NS\_OPTIONS

NS\_ENUM & NS\_OPTIONS是在iOS 6/ Mac OS X 10.8 中引入的，它是使用枚举型的一种更佳方式。

###NS_ENUM
    typedef NS_ENUM(NSInteger, UITableViewCellStyle) {
        UITableViewCellStyleDefault,
        UITableViewCellStyleValue1,
        UITableViewCellStyleValue2,
        UITableViewCellStyleSubtitle
    };
    
第一个参数是新类型的存储类型，而第二个参数则是新类型的名字。

###NS_OPTIONS
NS_OPTIONS则是为了方便使用位码而设置的。

    typedef NS_OPTIONS(NSUInteger, UIViewAutoresizing) {
        UIViewAutoresizingNone                      = 0,
        UIViewAutoresizingFlexibleLeftMargin        = 1 << 0,
        UIViewAutoresizingFlexibleWidth             = 1 << 1,
        UIViewAutoresizingFlexibleRightMargin       = 1 << 2,
        UIViewAutoresizingFlexibleTopMargin         = 1 << 3,
        UIViewAutoresizingFlexibleHeight            = 1 << 4,
        UIViewAutoresizingFlexibleBottomMargin      = 1 << 5
    };

它的语法看上去和NS_ENUM完全一样，不过编译器知道这些选项可以用位运算来组合。

