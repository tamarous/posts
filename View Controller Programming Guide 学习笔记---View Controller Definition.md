本文是阅读Apple官方文档《View Controller Programming Guide》时所记录的学习笔记。欢迎指出其中的错误。

## View Controller Definition
### 定义子类
在开发时我们通常会使用自定义的UIViewController子类视图控制器来显示应用的内容。那么在定义子类时应该考虑以下几点：

#### 定义UI
苹果推荐使用storyboard技术来管理和设计应用程序的UI界面。使用storyboard可以完成下面这些任务：

* 添加，管理和配置视图控制器的视图
* 连接outlet和动作
* 在视图控制器之间创建联系和segue(过渡）
* 为不同的尺寸类别定制布局和视图
* 向视图添加手势识别器来处理用户交互。

#### 处理用户交互
应用的响应者接收事件并进行适当的处理。虽然视图控制器也是一种响应者，但是它们一般不直接处理触摸事件。它们一般使用下列方式来处理事件：
    
1. 通过定义行为方法来处理高级的事件。

2. 观察和接收系统或者其他对象发送给它们的通知。

3. 充当其他对象的数据源或者委托。

####运行时显示视图

storyboard使得加载和显示视图的内容变得十分简单。当需要时，UIKit会自动从storyboard中加载视图。加载过程中UIKit会按顺序执行下列操作：

1. 使用storyboard中的信息来初始化视图。

2. 连接所有的outlet和action。

3. 将视图控制器的view属性设置为根视图

4. 调用视图控制器的awakeFromNib方法。调用了这个方法后，视图控制器的特征集合还是空的，视图还不位于它们最终的位置上。

5. 调用视图控制器的viewDidLoad方法。（使用这个方法来添加或删除视图、修改布局限制、加载数据等）

在加载完视图之后和在屏幕上显示视图之前，UIKit提供了额外的机会来准备这些视图。这即是说，UIKit会按下列顺序执行任务：

1. 调用视图控制器的viewWillAppear:方法来让它知道视图已经准备好显示在屏幕上了。

2. 更新视图的布局。

3. 在屏幕上显示视图。

4. 在屏幕上显示视图后，调用viewDidAppear:方法。


####管理视图布局

当视图的位置和大小改变时，UIKit会更新视图的布局信息，同时在某几个关键点发出通知，来让你完成一些额外的与布局有关的任务。对于每一个受影响的视图控制器，UIKit执行了下面这些任务：

1. 根据需要来更新视图控制器和它的视图的特征集合。

2. 调用视图控制器的viewWillLayoutSubviews方法。

3. 调用当前UIPresentationController对象的containerViewWillLayoutSubviews方法。

4. 调用视图控制器根视图的layoutSubviews方法。

    这个方法的默认实现是使用现有的限制来计算新的布局信息，然后遍历整个视图等级结构，为每个子视图调用layoutSubviews方法。

5. 将计算出的布局信息应用到视图上。

6. 调用视图控制器的viewDidLayoutSubviews方法。

7. 调用当前UIPresentationController对象的containerViewDidLayoutSubviews方法。

视图控制器可以使用viewWillLayoutSubviews和viewDidLayoutSubviews方法来执行一些影响布局过程的额外更新。在布局前，你可以添加或移除视图、更新视图的大小和位置信息、更新限制、或者更新其他与视图有关的属性。在布局后，你可以重新加载表数据，更新其他视图的内容，或者对视图的大小和位置做出最终的调整。

#### 有效管理内存

分配和释放内存的地方：

|Task|Methods|
|:---:|:----:|
|为视图控制器要使用到的数据结构分配内存|初始化方法|
|分配或者加载要显示在视图中的数据|viewDidLoad|
|响应内存低通知|didReceiveMemoryWarning|
|释放视图控制器中使用到的数据结构的内存|dealloc|

### 实现自定义的容器视图控制器
在实现自定义的容器视图控制器时，应该遵循下面几条规则：

#### 设计一个自定义的内容视图控制器
在设计你自己的内容视图控制器时，需要仔细考虑它和被包含的视图控制器之间的关系。你可以问自己以下问题：

1. 这个容器，和它的子视图控制器分别扮演着什么样的角色呢？
2. 多少个子视图控制器被同时显示？
3. 兄弟视图控制器之间的关系是怎么样的呢？
4. 子视图控制器怎么添加进或者从容器中删除？
5. 子视图控制器的大小和位置能更改吗？这些更改发生在什么样的条件下？
6. 容器自己会提供一些装饰性的或者与导航相关的视图吗？
7. 容器和子视图控制器之间如何进行通信？除了UIViewController所提供的那些外，容器还需要告知子视图控制器其他事件吗？

#### 在Interface Builder中为视图控制器配置正确的类型

#### 实现自定义的内容视图控制器
##### 向内容中添加子视图控制器

1. 调用容器视图控制器的`addChildViewController:`方法。这个方法告知UIKit现在你的自定义容器开始管理子视图控制器的视图。
2. 将子视图控制器的根视图加入容器的视图结构结构中。
3. 添加用来管理子视图控制器的根视图的大小和位置的限制。
4. 调用子视图控制器的`didMoveToParentViewController:`方法。


        - (void) displayContentController:(UIViewController *)content
        {
                [self addChildViewController:content];
                content.view.frame = [self frameForContentController];
                [self.view addSubview:self.currentClientView];
                [content didMoveToParentViewController:self];
        }

##### 移除子视图控制器
1. 调用子视图控制器的`willMoveToParentViewController:`方法，传值为nil。
2. 移除子视图控制器的根视图上的所有限制。
3. 将子视图控制器的根视图从容器的视图层次结构中移除。
4. 调用子视图控制器的`removeFromParentViewController`方法。

        - (void) hideContentController:(UIViewController    *)content 
        {
                [content willMoveToParentViewController:nil];
                [content.view removeFromSuperview];
                [content removeFromParentViewController];
        }

##### 子视图控制器间的过渡
    - (void) cycleFromViewController:(UIViewController *) oldVC
                        toViewController:(UIViewController *) newVC
        {
            // Prepare the two view controllers for the change.
            [oldVC willMoveToParentViewController:nil];
            [self addChildViewController:newVC];
        
            // Get the start frame of the new view controller and the end frame 
            // for the old view controller. Both rectangles are offscreen.
            newVC.view.frame = [self newViewStartFrame];
            CGRect endFrame = [self oldViewEndFrame];
        
            [self transitionFromViewController: oldVC toViewController:newVC
                    duration:0.25 options:0
                    animations:^{
                        newVC.view.frame = oldVC.view.frame;
                        oldVC.view.frame = endFrame;
                        }
                    completion:^{
                        [oldVC removeFromParentViewController];
                        [newVC didMoveToParentViewController:self];
                        }];
            
        }

##### 管理子视图控制器的外观更新

当将一个子视图控制器添加到容器视图控制器后，容器会自动将与外观有关的消息转发到此子视图控制器上。
你可以重载下面的函数来阻止此过程。

    - (BOOL) shouldAutomaticallyForwardApperanceMethods             
    {
       return NO;
    }
    
#### 构建一个容器视图控制器的建议

当实现自定义容器视图控制器时，需要考虑以下的建议：

1. 只访问子视图控制器的根视图
2. 子视图控制器对容器视图控制器了解得越少越好
3. 一开始时，使用普通的视图来测试你的容器是否正确工作。

#### 将一些特性委托给子视图
容器可以让子视图来决定它的状态栏的风格，只要在容器视图控制器中重载`childViewControllerForStatusBarStyle`和`childViewControllerForStatusBarHidden`这两个方法的一个或全部即可。


### 保存和恢复状态
保存应用的视图控制器的状态需要以下几步：

1. (必须)为你想保存状态的视图控制器分配一个恢复标识符。
2. (必须)告诉iOS在启动时如何创建和定位新的视图控制器对象。
3. (可选)对于每一个视图控制器，将能够把视图控制器恢复至原始状态的配置数据存储起来。

####标记需要存储的视图控制器
UIKit仅仅保存你告诉它去保存的视图控制器。每个视图控制器都有一个`restorationIdentifier`，这个值在默认情况下是nil的。将这个值设为一个特定的值时，UIKit就意识到应该保存这个视图控制器了。当给一个视图控制器分配保存标识符时，它的所有父视图控制器也需要分配标识符。在保存的过程中，UIKit从window的根视图控制器开始，沿着视图控制器的继承关系往下走，如果一个视图控制器没有保存标识符，那么这个控制器视图和它的子视图控制器都不会得到保存。

####选择有效的保存标识符
保存标识符通常是视图控制器的类名。如果一个类在很多地方使用，那么就需要更复杂更有意义的标识符。总之，标识符必须是独特的。

#### 保存类的视图
一些视图拥有一些额外的状态信息，这些信息为此视图所独有，而它的父视图控制器则不知道这些状态。举例来说，一个滚动视图拥有一个滚动位置，而这个量是滚动视图的视图控制器看不到的，所以滚动视图自己需要负责来保存这个信息。

要保存一个视图的状态，需要以下几步：

1. 给视图的`restorationIdentifier`属性分配一个有效的字符。
2. 确保这个视图所在的视图控制器拥有保存标识符。
3. 对于表视图和集合视图，分配一个遵循`UIDataSourceModelAssociation`协议的数据源。


#### 在启动时恢复视图控制器
在应用启动时，UIKit尝试着将应用恢复至它以前的状态，此时它需要应用创建或者定位组成之前用户界面的视图控制器对象。UIKit会按照以下顺序来定位视图控制器：

1. 如果视图控制器拥有一个恢复类，那么UIKit要求这个类来提供视图控制器。UIKit会调用这个类的`viewControllerWithRestorationIdentifierPath:Coder`方法来获取视图控制器。如果这个方法的返回值是nil，那么应用被假定成不想重建视图控制器，UIKit也会停止寻找过程。
2. 如果视图控制器没有恢复类，那么UIKit会要求应用委托来提供这个视图控制器。UIKit会调用应用委托的`application:viewControllerWithRestorationIdentifierPath:coder:`方法来寻找没有恢复类的视图控制器。如果这个方法返回nil，那么UIKit会隐式地寻找这个视图控制器。
3. 如果存在一个有正确恢复路径的视图控制器，那么UIKit就使用这个对象。
4. 如果视图控制器原来是从一个故事板文件中加载的，那么UIKit使用保存下来的故事板信息来定位和创建它。

给视图控制器分配一个恢复类可以阻止UIKit隐式寻找这个视图控制器。使用恢复类可以让你更好地去控制是否真的想要创建一个视图控制器。举例来说，恢复类的`viewControllerWithRestorationIdentifierPath:Coder`如果返回nil，那么这个视图控制器就不会被重新创建。

当使用恢复类时，你的`viewControllerWithRestorationIdentifierPath:Coder`应该创建一个新的类实例，执行最小的初始化，然后返回该结果。

    + (UIViewController *) viewControllerWithRestorationIdentifierPath:(NSArray *)
        identifierComponents coder:(NSCoder *)coder {
            MyViewController *vc;
            UIStoryboard *sb = [coder decodeObjectForKey:UIStateRestorationViewControllerStoryboardKey];
            if  (sb) {
                vc = (PushViewController *)[sb instantiateViewControllerWithIdentifier:@"MyViewController"];
                vc.restorationIdentifier = [identifierComponents lastObjects];
                vc.restorationClass = [MyViewController class];
            }
    
            return vc;
        }
        
当通过手工重新创建视图控制器时，重新确定恢复标识符和恢复类是一个较好的习惯，最简单的方法就是从identifierComponents数组中拿出最后一个并且将它赋给视图控制器。

#### 对视图控制器的状态进行编码和解码
对于每一个要保存的对象，UIKit调用这个对象的`encodeRestorableStateWithCoder:`方法来给它一次机会保存下自己的状态。在恢复的过程中，UIKit调用对应的`decodeRestorableStateWithCoder:`方法来对状态进行解码，并重新赋给那个对象。实现这些方法是可选的，也是推荐的。你必须使用这些方法来保存下面这些类型的信息：

* 正在被展示的数据的饮用
* 容器视图控制器的子视图控制器的饮用
* 当前选中情况的饮用
* 对于拥有一个用户可配置视图的视图控制器，保存下那个可配置视图的信息。


         - (void) encodeRestorableStateWithCoder:(NSCoder *)coder {
            [super encodeRestorableStateWithCoder:coder];
            [coder encodeInt:self.number forKey:MyViewControllerNumber];
        }
    
        - (void) decodeRestorableStateWithCoder:(NSCoder *)coder {
            [super decodeRestorableStateWithCoder:coder];
            self.number = [coder decodeIntForKey:MyViewControllerNumber];
        }
    
#### 保存和恢复视图控制器的一些tips

* 记住不需要保存所有的视图控制器
* 避免在恢复的过程中改变视图控制器的类名
* 恢复系统期待你按原来的方式来使用恢复出来的视图控制器。

 
 