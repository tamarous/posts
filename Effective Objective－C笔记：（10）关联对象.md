#Effective Objective－C笔记：（十）关联对象

在iOS开发时，UIAlerView经常被用到。该类提供了一种标准视图，可向用户展示警告信息。当用户点击弹出的按钮来关闭该视图时，需要实现UIAlertView的委托协议来处理此动作。但是一般说来，我们是将创造警告视图和处理按钮动作分开的。所以代码可能会被放在不同的位置，显得比较凌乱。

一般说来，UIAlertView是这样写的：

    - (void) askUserQuestion {
        UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Question"
        message:@"What do you want to do?"
        delegate:self
        cancelButtonTitle:@"Cancel"
        otherButtonTitle:@"Continue",nil];
        [alert show];
    }

    #pragma mask - UIAlertViewDelegate protocol method
    - (void)alertView:(UIAlertView *)alertView 
    clickedButtonAtIndex:(NSInteger)buttonIndex {
        if(buttonIndex == 0) {
            [self doCancel];
        } else {
            [self doContinue];
        }
    }


如果要在这个文件中处理多个AlertView，那么就必须在delegate方法中对传入的alertView进行判断，然后调用相应的逻辑。有一种更好的方法是在创建AlertView的时候，直接把处理每个按钮的逻辑给写好，这样会简单很多。这时就可以使用**关联对象**这一手段了。

使用关联对象时，需要导入一个特殊的头文件：

    #import <objc/runtime.h>
    
    static void *EOCMyAlertViewKey = @"EOCMyAlertViewKey";

    - (void) askUserAQuestion {
        UIAlertView *alert = [[UIAlertView alloc]
        initWithTitle:@"Question"
        message:@"What do you want to do?"
        delegate:self
        cancelButtonTitle:@"Cancel"
        otherButtonTitles:@"Continue",nil];

        void (^block)(NSInteger) = ^(NSInteger buttonIndex)         {
            if (buttonIndex == 0) {
                [self doCancel];
            }
            else {
                [self doContinue];
            }
        };

        objc_setAssociatedObject(alert,
                                EOCMyAlertView,
                                block,
                                OBJC_ASSOCIATION_COPY);
        [alert show];
    }

    - (void) alertView:(UIAlertView *)alertView clickedButtonAtIndex:(NSInteger)buttonIndex{
            void (^block)(NSInteger) = objc_getAssociatedObject(alertView,EOCMyAlertViewKey);
            
            block(buttonIndex);
    }
    

通过这种写法，创建警告视图和处理它的按钮动作的代码都放在一起了。这样读起来更加方便，别人在看你的代码时不需要在代码之间来回跳转了。但是block可能会捕获一些变量，因此可能会导致强引用环。所以使用时需要小心谨慎。

这段代码中的关键方法有两个：

    //此方法以给定的键和策略为某对象设置关联对象值
    void objc_setAssociatedObject(id object, void *key, id value, objc_AssociationPolicy policy)


    //此方法根据给定的键从某对象中获取相应的关联对象值
    void objc_getAssociatedObject(id object, void *key)
    
第一个方法中的objc_AssociatedPolicy是存储策略，允许的取值是如下的枚举值：

    OBJC_ASSOCIATION_ASSIGN
    OBJC_ASSOCIATION_RETAIN_NONATOMIC
    OBJC_ASSOCIATION_COPY_NONATOMIC
    OBJC_ASSOCIATION_RETAIN
    OBJC_ASSOCIATION_COPY
    
除了这两个方法外，还有一个方法用于移除一个指定对象的所有关联对象：

    void objc_removeAssociatedObjects(id object)
    




 
 

