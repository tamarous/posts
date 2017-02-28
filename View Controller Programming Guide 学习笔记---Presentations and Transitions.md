## 显示和过渡

### 显示一个视图控制器
当显示一个视图控制器时，被显示的视图控制器和原来的视图控制器就形成了一种关系：原来的视图控制器被称为`presenting view controller`，被显示的视图控制器被称为`presented view controller`。这种关系一直维持到presented view controller消失为止。

#### 呈现和过渡过程

##### 呈现方式
在显示视图控制器时，可以通过为视图控制器的modalPresentationStyle属性设置常量值来选择合适的外观风格。
###### 全屏呈现方式
全屏风格遮盖了整个屏幕，阻止用户和下面的内容进行交互。有三种全屏风格，分别是`UIModalPresentationFullScreen`, `UIModalPresentationPageSheet`和`UIModalPresentationFormSheet`。当设备处于水平正常环境下时，只有`UIModalPresentationFullScreen`会完全遮住屏幕，其余的两种只是遮住了部分屏幕。当设备处于水平紧凑环境下时，全屏外观会自动采用`UIModalPresentationFullScreen`并完全遮住屏幕。

下图显示了这三种风格各是什么样子。
![Full Screen presentation styles](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_PresentationStyles%20_fig_8-1_2x.png)

###### 弹出风格

###### 当前上下文风格
UIModalPresentationCurrentContext会遮住某个特定的视图控制器。只要将想要遮住的视图控制器的`definesPresentationContext`属性设置为YES即可。

定义了上下文的那个视图控制器同样可以定义呈现时的过渡动画效果。

上文中我们提到，UIKit会使用presented视图控制器的`modalTransitionStyle`来动画化该视图的过渡效果。如果展示上下文的视图控制器的`providesPresentationContextTransitionStyle`属性被设置为YES，那么UIKit就会转而使用该展示上下文的视图控制器的`modalTransitionStyle`属性。

###### 自定义呈现风格
UIModalPresentationCustom让我们可以使用自定义的风格来呈现一个视图控制器。注意，此处是自定义的呈现风格，而上文中是自定义过渡的风格。

如果想要使用这种风格，那么需要先子类化UIPresentationController，然后使用这个子类中的方法来动画化所有的自定义视图，并且设置要展示的视图控制器的位置和大小属性。

##### 过渡风格
过渡风格决定了显示一个presented视图控制器时的过渡动画的样式。对于内置的过渡风格，在为视图控制器的`modalTransitionStyle`属性赋值后，UIKit就会在显示此视图控制器时采用对应的过渡风格。

##### presenting和showing的对比
UIViewController提供了两种显示视图控制器的方法：

1. showViewController:sender:和showDetailViewController:sender:提供了呈现视图控制器的最灵活、适应性最好的方法。
2. presentViewController:animated:completion:方法总是**模态**地呈现视图控制器。


#### 呈现一个视图控制器
下面是几种**初始化**视图控制器的展示过程的方法。

* 使用segue。segue使用你在Interface Builder中设定的一些信息来初始化和显示视图控制器。
* 使用showViewController:sender:和showDetailViewController:sender:方法来显示视图控制器。
* 调用presentViewController:animated:completion:方法来**模态**地显示视图控制器。

##### Show视图控制器
当使用showViewController:sender:和showDetailViewController:sender:方法时，在屏幕上显示一个新视图控制器的过程如下：

1. 创建想要呈现的视图控制器。
2. 将新视图控制器的modalPresentationStyle设置为合适的呈现风格。
3. 将新视图控制器的modalTransitionStyle设置为合适的过渡风格。
4. 调用当前视图控制器的showViewController:sender和showDetailViewController:sender。

UIKit将对showViewController:sender和showDetailViewController:sender调用转发至合适的presenting视图控制器。这个视图控制器可以决定如何最好地来执行呈现过程，以及按照需要来调整呈现和过渡的风格。


                
##### 模态地呈现视图控制器
当直接呈现一个视图控制器时，告诉UIKit你想要这个新视图控制器如何被显示，以及它应该怎样被动画至屏幕上。
1. 创建你想要呈现的视图控制器。
2. 将新视图控制器的modalPresentationStyle属性设置为想要的呈现方式。
3. 将新视图控制器的modalTransitionStyle属性设置为想要的动画方式。
4. 调用当前视图控制器的presentViewController:animated:completion:方法。

调用presentViewController:animated:completion:方法的视图控制器不一定是实际执行模态呈现过程的那个视图控制器。呈现风格决定了视图控制器该如何显示，也确定了presenting视图控制器所需要的一些特性。比如说，一个全屏的呈现必须是由一个全屏视图控制器来初始化的。如果当前的视图控制器不合适，那么UIKit就会沿着视图控制器的层次结构寻找，直到找到合适的。当模态呈现过程完成时，UIKit会更新受影响的视图控制器的presentingViewController和presentedViewController。

	- (void)add:(id)sender {
           
           RecipeAddViewController *addController = [[RecipeAddViewController alloc] init];
         
           addController.modalPresentationStyle = UIModalPresentationFullScreen;
           addController.transitionStyle = UIModalTransitionStyleCoverVertical;
           [self presentViewController:addController animated:YES completion: nil];
        }

##### 弹出窗口中展示视图控制器

弹出式的视图控制器需要进行一些额外配置才能够显示。在将显示风格设置为`UIModalPresentationPopover`后，要设置下列这些属性的值：

* 设置视图控制器的`preferredContentSize`为合适的尺寸。
* 通过设置视图控制器的`popoverPresentationController`属性，使用相应的`UIPopoverPresentationController`对象来设置弹出窗口的固定点。

#### 关闭一个presented视图控制器

为了清除一个presented视图控制器，可以调用presenting视图控制器的dismissViewControllerAnimated:completion:方法。你也可以调用presented视图控制器的此方法来清除它。如果你这样做的话，UIKit会自动将方法调用转发到presenting视图控制器。

在清除一个视图控制器之前，一定要存储所有重要的信息。

如果presented视图控制器需要返回数据给presenting视图控制器，那么可以使用`delegation`这种设计模式。


#### 显示一个定义在不同的故事板中的视图控制器

    UIStoryboard* sb = [UIStoryboard storyboardWithName:@"SecondStoryboard" bundle:nil];
    MyViewController* myVC = [sb instantiateViewControllerWithIdentifier:@"MyViewController"];
     
    // Configure the view controller.
     
    // Display the view controller
    [self presentViewController:myVC animated:YES completion:nil];          
    
应用程序中对于使用多少个故事板文件并没有要求。但是在某些情况下使用多个故事板可能会很有用。


### 使用Segue
segue在故事板文件中定义了两个视图控制器之间的过渡。segue的起始点可以是一个按钮、列表项或者是手势识别器，而segue的终止点则是想要显示的那个视图控制器。

#### 在视图控制器间创建segue
要在视图控制器间创建segue，只需要按住ctrl，然后从第一个视图控制器中的某个元素拖拽到第二个视图控制器上即可。当释放时，Interface Builder会提示你为segue选择一种合适的类型。下面是可供选择的segue类型：

|Segue type|Behavior|
|:---|:---|
|Show(Push)|这种segue会使用`showViewController:sender:`方法来显示目标视图控制器。对于大多数视图控制器来说，这种segue会在源视图控制器上模态地展示新视图控制器的内容。一些视图控制器会重载这个方法以实现不同的呈现方法。UIKit会使用`targetViewControllerForAction:sender:`方法来定位源视图控制器的位置|
|Show Detail(Replace)|这种segue使用目标视图控制器的`showDetailViewController`来显示它的内容。只当源和目标视图控制器都嵌在UISplitViewController中时，这种segue才是有意义的。使用它，UISplitViewController可以将第二个视图控制器替换为新内容。|
|Present Modally|这种segue会使用指定的显示和过渡风格**模态**地呈现新视图控制器。|
|Present as Popover|在水平常规的环境中，视图控制器会显示在一个弹出窗口中。在水平紧凑的环境中，视图控制器会使用一个全屏模态风格来显示。|

#### 在运行时修改segue的行为
下图显示了当一个segue被触发时究竟发生了什么。
![Displaying a view controller using a segue](https://developer.apple.com/library/iOS/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_displaying-view-controller-using-segue_9-4_2x.png)

在segue期间，UIKit会调用当前视图控制器的某些方法，你可以通过重载这些方法来影响segue的结果：
* `shouldPerformSegueWithIdentifier:sender:`提供了阻止一个segue发生的机会。如果这个方法返回NO，那么对应的segue就会失败，但是其他行为不受影响。
* 源视图控制器的`prepareForSegue:sender:`方法允许你将数据从源视图传递到目的视图控制器。

#### 创建一个unwind segue
unwind segue让你可以关闭已被展示的视图控制器。
要创建一个unwind segue，遵循如下步骤：
1. 选择那个在unwind segue结束后应该展示在屏幕上的视图控制器
2. 在所选择的那个视图控制器中定义一个unwind行为方法:
	> @IBAction func myUnWindAction(unwindSegue:UIStoryboardSegue)
	> -(IBAction) myUnWindAction(UIStoryboardSegue *)unwindSegue
3. 导航至初始化unwind segue的那个视图控制器
4. 按下ctrl，然后从初始化segue的那个按钮或是列表项，将箭头拖到视图控制器场景的Exit处
5. 从弹出的菜单中选择你在第二步中定义的unwind行为方法。

#### 用代码的方式初始化一个segue
		
```
- (void) orientationChanged:(NSNotification *)notification {
		UIDeviceOrientation deviceOrientation = [UIDevice currentDevice].orientation;
		if	(UIDeviceOrientationIsLandscaped(deviceOrientation) && ! isShowingLandscapeView) {
			[self performSegueWithIdentifier:@"DisplayAlternateView" sender:self];
			isShowingLandscapeView = YES;
		}
	}
```
####创建一个自定义的segue
#####segue的生命周期
`segue对象`是UIStoryboardSegue或者它的子类的实例对象。应用从不直接创建segue对象，UIKit会在一个segue被触发时自动创建它们。
1. 要被显示的视图控制器被创建并被初始化了。
2. segue对象被创建，并且它的`initWithIdentifier:source:destination:`方法被调用了。
3. `presenting view controller`的`prepareForSegue:sender:`方法被调用。
4. segue对象的perform方法被调用。
5. segue对象的引用被释放。

#####实现一个自定义segue
要实现一个自定义的segue，需要让一个类继承自UIStoryboardSegue，然后实现下列方法：
* 重载`initWithIdentifier:source:destination:`方法，并且使用这个方法来初始化自定义的segue对象。
* 实现perform方法并且用该方法来配置过渡动画。


```
- (void) perform {
		[[self sourceViewController] presentViewController:[self destinationViewController] animated:NO completion:nil];
	}
```

###自定义过渡动画
#### transition delegate
Transition delegate遵循UIViewControllerTransitionDelegate，它的职责是为UIKit提供以下对象：
* Animator object负责创建动画。委托中可以为一个视图控制器提供不同的Animator object。Animator object遵循UIViewControllerAnimatedTransitioning委托。
* 交互式的animator对象使用触摸事件或者手势识别器来驱动自定义动画的计时。它遵从UIViewControllerInteractiveTransitioning协议。创建一个交互式animator objects最简单的方法是声明一个继承自UIPercentDrivenInteractiveTransition的类，然后向子类中添加事件处理方法。
* 外观控制器是当一个视图控制器在屏幕上时管理它的展示风格。

将一个transition delegate赋值给视图控制器的`transitionDelegate`属性会告知UIKit你想要执行自定义的展示或过渡。委托可以决定要提供哪些对象，比如说如果没有提供animator对象，那么UIKit就会使用视图控制器的modalTransitionStyle的标准过渡动画。

#### 自定义的动画顺序
如果被显示的视图控制器的transitionDelegate属性包含了一个有效的值，那么UIKit会使用你提供的自定义animator对象来显示那个视图控制器。在准备显示的时候，UIKit会调用transition delegate的
`animationControllerForPresentedController:presentingController:sourceController:`方法来获取自定义的animator object。如果这个object可获得，那么UIKit会执行下列步骤：
1. UIKit调用transition delegate的interactionControllerForPresentation:方法来看是否存在交互式的animator objects，如果这个方法返回nil，那么UIKit就会执行没有交互的动画。
2. UIKit调用animator object的transitionDuration:方法来得到动画时长。
3. UIKit调用合适的方法来开始动画：
	* 对于非交互式的动画，UIKit调用animator object的animateTransition:方法
	* 对于交互式的动画，UIKit调用交互式animator object的startInteractiveTransition:方法。
4. UIKit等待一个animator object调用当前上下文的transition object的completeTransition:方法

当关闭一个视图控制器时，UIKit调用transition delegate的animationControllerForDismissedController:方法，剩下的步骤与上文一只。

#### 过渡上下文对象
在过渡动画开始前，UIKit会创建一个过渡动画上下文对象，并且告知它该如何执行这些动画。过渡上下文对象是你的代码的一个重要组成部分，它实现了UIViewControllerContextTransitioning协议，并且存储了对视图控制器和视图的引用。它同时也记录了你该如何执行过渡，动画是否是交互的等信息。animator object需要这些信息来执行实际的动画。

下图解释了过渡上下文对象是如何和其他对象产生联系的。
![The transitioning context object](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_transitioning-context-object_10-2_2x.png)


#### 过渡协调者
对于内置的和自定义的过渡效果，UIKit会创建一个过渡协调者对象来便于执行可能需要的额外动画。除了显示和关闭视图控制器时会执行动画以外，当界面发生旋转或者视图控制器的frame变化时，也有过渡发生。所有的这些过渡都代表了对视图体系的改变。过渡协调者就是一种能记录这些变化并且动画化内容的方法。可以通过受到改变影响的视图控制器的transitionCoordinator属性来获得这个过渡协调者。过渡协调者仅仅在过渡阶段中存在。

下图显示了在一个显示过程中，过渡协调者和视图控制器之间的关系。使用过渡协调者来获得过渡的信息，并且注册想要和过渡一起执行的动画block。
![The transition coordinator objects](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_transition-coordinator-objects_10-3_2x.png)

#### 使用自定义动画来显示视图控制器
1. 创建想要展示的视图控制器
2. 创建自定义的transitioning delegate，将它赋值给视图控制器的transitioningDelegate属性。
3. 调用presentViewController:animated:completion:来显示视图控制器。

当在第三步中调用了presentViewController:animated:completion:方法后，UIKit会初始化显示过程。显示过程会在下一次run loop开始，直到自定义的animator调用completeTransition:方法时才结束。

#### 实现transitioning delegate
transitioning delegate的作用就是创建和返回自定义的对象。
```
- (id<UIViewControllerAnimatedTransitioning>)
	animationControllerForPresentedController:(UIViewController *)presented
	presentingController:(UIViewController *)source {
	MyAnimator *animator = [[MyAnimator alloc] init];
	return animator;
}
```
#### 实现animator objects
Animator object遵循了`UIViewControllerAnimatedTransitioning`协议。它的关键方法是animateTransition:方法，此方法是创建实际动画的方法。动画过程被分为了以下几步：
1. 得到动画参数
2. 使用Core Animation或者UIView动画方法来创建动画。
3. 清理并完成动画。
##### 得到动画参数
animateTransition:的参数transitionContext包含了执行动画时所需的数据。

> \- (void) animateTransition:(id\<UIViewControllerContextTransitioning\>)transitionContext

* 调用两次viewControllerForKey:方法来获得过渡过程中的from和to视图控制器。不要认为你自己知道过渡发生前后的视图控制器究竟是什么。
* 调用viewForKey:方法来得到要添加或者移除的视图。
* 调用finalFrameForViewController:方法来得到被添加或者移除的视图的最终的frame rectangle。

transitionContext使用from和to这种命名系统来确定一场过渡中涉及的视图控制器、视图和frame rectangle。"from"总是是一次过渡开始时出现在屏幕上的视图控制器，而to是过渡结束时才可见的视图控制器。

![The from and to objects](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/Art/VCPG_from-and-to-objects_10-4_2x.png)

##### 创建过渡动画
展示动画：
* 使用viewControllerForKey:和viewForKey:方法来取得过渡中涉及到的类和视图。
* 设置to视图的开始位置。设置其他必要的属性为它们的初始值。
* 使用transitionContext的finalFrameForViewController:来获得to视图的终止点。
* 将to视图作为子视图加进容器视图。
* 创建动画
	1. 在动画的block中，将to视图动画化至它在容器视图中最后的位置。将其他属性也设置为它们的终止值。
	2. 在完成的block中，调用completeTransition:方法，并执行其他清理过程。

关闭动画
* 使用viewControllerForKey:和viewForKey:方法来获得过渡中涉及到的视图控制器和视图。
* 计算from视图的最终位置。
* 将to视图作为子视图加进容器视图。
* 创建动画
	1. 在动画的block中，将from视图动画至它在容器视图中最后的位置。将其他属性也设置为它们的终止值。
	2. 在完成的block中，将from视图从视图层次中移除，然后调用completeTransition:方法，并执行其他清理过程。

	
```
- (void) animateTransition:(id<UIViewControllerContextTransitioning>)transitionContext {

    //Get the set of relevant objects.
    UIView *containerView = [transitionContext containerView];
    UIViewController *fromVC = [transitionContext viewControllerForKey:UITransitionContextFromViewControllerKey];
    UIViewController *toVC = [transitionContext viewControllerForKey:UITransitionContextToViewControllerKey];

    UIView *toView = [transitionContext viewForKey:UITransitionContextToViewKey];
    UIView *fromView = [transitionContext viewForKey:UITransitionContextFromViewKey];

    // Set up some variables for the animation.
    CGRect containerFrame = containerView.frame;
    CGRect toViewStartFrame = [transitionContext initialFrameForViewController:toVC];
    CGRect toViewFinalFrame = [transitionContext finalFrameForViewController:vc];
    CGRect fromViewFinalFrame = [transitionContext finalFrameForViewController:fromVC];

    // Set up the animation parameters.
    if (self.presenting) {
        //Modify the frame of the presented view so it starts 
        // offscreen at the lower-right corner of the container.
        toViewStartFrame.origin.x = containerFrame.size.width;
        toViewStartFrame.origin.y = containerFrame.size.height;
    }
    else {
        // Modify the frame of the dismissed view so it ends in the 
        // lower-right corner of the container view.
        fromViewFinalFrame = CGRectMake(containerFrame.size.width,
        containerFrame.size.height,
        toView.frame.size.width,
        toView.frame.size.height);
    }

    // Animate using the animator's own duration value.
    [UIView animateWithDuration:[self transitionDuration:transitionContext]
                     animations:^{
                         if (self.presenting) {
                             // Move the presented view into position.
                             [toView setFrame:toViewFinalFrame];
                         }
                         else {
                             // Move the dismissed view offscreen.
                             [fromView setFrame:fromViewFinalFrame];
                         }
                     }
                     completion:^(BOOL finished){
                         BOOL success = ![transitionContext transitionWasCancelled];
 
                         // After a failed presentation or successful dismissal, remove the view.
                         if ((self.presenting && !success) || (!self.presenting && success)) {
                             [toView removeFromSuperview];
                         }
 
                         // Notify UIKit that the transition has finished
                         [transitionContext completeTransition:success];
                     }];

}

```
##### 在动画后进行清理
在过渡动画结束后，调用completeTransition:方法非常重要。调用这个方法告诉UIKit过渡结束了，可以开始使用presented view controller了。同时其他completion handler也收到了通知，比如presentViewController:animated:completion:方法和animator的animationEnded:方法。最好的调用completeTransition:方法的地方是在动画block的completion handler中。

#### 向过渡添加交互性
让动画效果变得可交互的最简单的方法就是使用一个`UIPercentDrivenInteractiveTransition`对象。这个对象和现有的animator 对象们一起工作，使用你提供的已完成百分比的值，控制着它们的计时。你所要做的就是设置好用来更新完成百分比的事件处理代码，并且当新事件来临时更新这个值。

你可以子类或者不子类化这个UIPercentDrivenInteractiveTransition。如果你写了一个子类，那么在子类的init或者startInteractiveTransition:方法中一次性设置好事件处理代码。然后，调用此事件处理代码来计算一个新的完成百分比值，再调用updateInteractiveTransition:方法。当事件处理代码认为过渡结束时，调用finishInteractiveTransition方法。


```
- (void) startInteractiveTransition:(id<UIViewControllerContextTransitioning>)transitionContext {
    [super startInteractiveTransition:transitionContext];
    self.contextData = transitionContext;
    self.panGesture = [UIPanGestureRecognizer alloc] initWithTarget:self action:@selector(handleSwipeUpdate:)];
    self.panGesture.maximumNumberOfTouches = 1;

    UIView * container = [transitionContext containerView];
    [container addGestureRecognizer:self.panGesture];
}

- (void) handleSwipeUpdate:(UIGestureRecognizer *) gestureRecognizer {
    UIView* container  = [self.contextData containerView];

    if (gestureRecognizer.state == UIGestureRecognizerStateBegan) {
        [self.panGesture setTranslation:CGPointMake(0,0) inView:container];
    }
    else if (gestureRecognizer.state == UIGestureRecognizerStateChanged) {
        CGPoint translation = [self.panGesture translationInView:container];
        CGPoint percentage = fabs(translation.y/CGRectGetHeight(container.bounds));
        [self updateInteractiveTransition:percentage];
    }
    else if (gestureRecognizer.state == UIGestureRecognizerStateEnded) {
        [self finishInteractiveTransition];
        [[self.contextData containerView] removeGestureRecognizer:self.panGesture];
    }
}
```

####创建和过渡一起运行的动画
过渡中的视图控制器可以在显示或过渡动画之上展示额外的动画效果。例如说，presented view controller可以在过渡过程中给自己的视图层次加上动画。任何对象都可以创建动画，只要这个对象可以访问presenting或者presented 视图控制器的transitionCoordinator属性。
为了创建动画，调用transitionCoordinator的animateAlongsideTransition:completion:或者animateAlongsideTransitionInView:animation:completion:方法。

#### 使用外观控制器
对于自定义的展示过程，你可以提供自己的外观控制器来给presented view controller设置自定义的外观。外观控制器管理着除了视图控制器和它的视图意外的一切自定义chrome。

#### 创建自定义的外观控制器
当你显示一个外观风格是UIModalPresentationCustom的视图控制器时，UIKit会寻找一个自定义的外观控制器来管理这个显示过程。当显示过程在处理中时，UIKit调用外观控制器的方法，给予它机会来设置自定义的视图，然后将它们动画至合适的位置。
外观控制器和animator对象一起工作来完成整个过渡过程。animator对象将视图控制器的内容动画至屏幕上，而外观控制器处理所有其他事情。通常，外观控制器会动画它自己的视图，但是你也可以重载外观控制器的presentedView方法，让animator对象动画所有的或者一部分视图。


在显示过程中，UIKit：
1. 调用transitioning delegate的presentationControllerForPresentedViewController:presentingViewController:sourceViewController:方法来获取自定义的外观控制器。
2. 询问transition delegate，以提供animator和互动式animator。
3. 调用外观控制器的presentationTransitionWillBegin方法。你应该重载这个方法来向视图层次添加自定义视图，然后为这些视图配置动画。
4. 从外观控制器获得presentedView。通常这个方法会返回presented view controller的根视图。你的外观控制器可以用一个自定义的背景视图来替换那个根视图。
5. 执行过渡动画。过渡动画包括了由animator对象创建的和其他的一些你设置的以和主要动画一起运行的动画。在动画过程中，UIKit调用了外观控制器的containerViewWillLayoutSubviews和containerViewDidLayoutSubviews方法，因此你可以在此调整自定义视图的布局。

在关闭过程中，UIKit：
1. 从当前的可见视图控制器上可到自定义的外观视图控制器。
2. 询问transition delegate，以提供animator和互动式animator。
3. 调用外观控制器的dismissalTransitionWillBegin方法。你应该重载这个方法来向视图层次添加自定义视图，然后为这些视图配置动画。
4. 从外观控制器上得到已经准备好的presentedView。
5. 执行过渡动画。
6. 当过渡动画结束时调用dismissalTransitionDidEnd:方法。

在显示过程中，外观控制器的frameOfPresentedViewInContainerView和presentedView方法可能被多次调用，所以你的实现应该较快返回。同时，你的presentedView方法实现不应该尝试设置视图层次。当这个方法被调用时视图层次就应该被设置好了。

#### 创建自定义的外观控制器
要实现自定义的外观风格，子类化UIPresentaionController，然后添加代码来创建视图和动画。当创建自定义的外观控制器时，考虑下面几个问题：
* 你想添加什么视图？
* 你想怎么样在屏幕上为额外的视图添加动画？
* presented视图控制器的尺寸应该是什么样的？
* 外观控制器应该怎么样适应水平普通和水平紧凑的尺寸类别？
* 当显示过程结束时，presenting视图控制器的视图应该被移除吗？

#### 设置presented视图控制器的frame

```
- (CGRect) frameOfPresentedViewInContainerView {
    CGRect presentedViewFrame = CGRectZero;
    CGRect containerBounds = [[self containerView] bounds];

    presentedViewFrame.size = CGSizeMake(floorf(containerBounds.size.width/2.0),containerBounds.size.height);
    presentedViewFrame.origin.x = containerBounds.size.width-presentedViewFrame.size.width;
    return presentedViewFrame;
}
```
##### 管理和动画化自定义视图
自定义的外观通常包含着向被显示的内容上添加自定义视图。可以使用它们来添加实际的行为。比方说，一个背景视图可以和手势识别器一起合作来追踪被显示区域边界之外的某些行为。

外观视图控制器要负责创建和管理和它的外观有关的一切自定义视图。通常，你可以在视图控制器的初始化过程中创建自定义视图。


```
- (instancetype) initWithPresentedViewController:(UIViewController *)presentedViewController 
    presentingViewController:(UIViewController *)presentingViewController {
    self = [super initWithPresentedViewController:presentedViewController presentingViewController:presentingViewController];
    if  (self){
        self.dimmingView = [[UIView alloc] init];
        [self.dimmingView setBackgroundColor:[UIColor colorWithWhite:0.0 alpha:0.4]];
        [self.dimmingView setAlpha:0.0];
    }
    return self;
}
```
创建完自定义视图后，可以使用presentingTransitionWillBegin方法来将自定义视图动画至屏幕上。在这个方法中，配置好自定义视图，然后将它们加入到容器视图中。使用过渡协调者或者是presenting/presented视图控制器来创建动画。不要在这个方法中修改presented视图控制器的视图。animator对象负责将presented视图控制器动画至从frameOfPresentedViewInContainerView方法中返回的frame矩形。


```
- (void) presentationTransitionWillBegin {
    UIView *containerView = [self containerView];
    UIView *presentedViewController = [self presentedViewController];

    [[self dimmingView] setFrame:[containerView bounds]];
    [[self dimmingView] setAlpha:0.0];
    [containerView insertSubView:[self dimmingView] atIndex:0];
    if  ([presentedViewController transitionCoordinator]) {
        [[presentedViewController transitionCoordinator] 
        animateAlongsideTransition:^(id<UIViewControllerTransitionCoordinatorContext>context){
            [[self dimmingView] setAlpha:1.0];
        } completion:nil];
    }
    else {
        [[self dimmingView] setAlpha:1.0];
    }
}
```

在显示最后，使用presentationTransitionDidEnd:方法来处理由于显示被取消而引起的清理工作。
交互式animator对象可能在边缘条件不满足时取消一个过渡。当这种事情发生时，UIKit以NO为参数调用presentationTransitionDidEnd:方法。当发生取消过渡的事情时，移除添加的所有自定义视图，将其他视图重置为它们之前的配置。


```
- (void) presentationTransitionDidEnd:(BOOL) completed {
    if (!completed) {
        [self.dimmingView removeFromSuperview];
    }
}
```

当视图控制器被关闭时，使用dismissalTransitionDidEnd:方法来从视图层次结构中移除所有的自定义视图。如果想为移除过程也添加上动画，可以在dismissalTransitionDidEnded:方法中设置这些动画。

```
- (void) dismissalTransitionWillBegin {
    if([[self presentedViewController] transitionCoordinator]) {
        [[[self presentedViewController] transitionCoordinator] 
        animateAlongsideTransition:^(id<UIViewControllerTransitionCoordinator> context){
            [[self dimmingView] setAlpha:0.0];
        } completion:nil];
    }
    else {
        [[self dimmingView] setAlpha:0.0];
    }
}

- (void) dismissalTransitionDidEnd:(BOOL) completed {
    if (completed) {
        [self.dimmingView removeFromSuperview];
    }
}
```

##### 给UIKit提供自定义外观控制器
当展示一个视图控制器时，遵循下列步骤以使用自定义的外观控制器。
* 设置presented视图控制器的modalPresentationStyle属性为UIModalPresentationCustom。
* 设置presented视图控制器的transitionDelegate属性为一个transitioning delegate。
* 实现transitioning delegate的presentationControllerForPresentedViewController:presentingViewController:sourceViewController:方法

第三步中如此长的方法是为UIKit提供它所需要的presentation controller。

```
- (UIPresentationController *)presentationControllerForPresentedViewController:
        (UIViewController *)presented
        presentingViewController:(UIViewController *)presenting
        sourceViewController:(UIViewController *)source {
        MyPresentationViewController *presentation = [MyPresentation initWithPresentedViewController:presented presentingViewController:presenting];
        return presentation;
        }
```




