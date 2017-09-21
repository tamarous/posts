##译：CALayer Tutorial：Getting Started（上）

本篇文章翻译自发表在Raywenderlich上的一篇教程: [CALayer Tutorial：Getting Started](https://www.raywenderlich.com/90488/calayer-in-ios-with-swift-10-examples)，在不影响文章完整性的前提下对原文有所删减。

你可能已经知道，你在一个iOS 应用中看到的一切元素都是视图，无论是按钮、表视图还是滑块，甚至是包含了其他视图的父视图。
但是你可能不知道，在每个视图后，还藏着一个叫做layer的类，准确的来说，这个类的名字叫做CALayer。
下面通过一个项目来对这个类的知识进行介绍。

让我们打开Xcode，

1. 创建一个Single View Application，编程语言选择Swift，不要勾选单元测试、Core Data和版本管理。
2. 项目创立后，选择Main.storyboard。
3. 在菜单栏中，选择Editor\Canvas\Show Bounds Rectangle。这时我们可以看到Main.storyboard中的视图周围出现了蓝色的轮廓线。
4. 向视图控制器中拖入一个视图，然后选中它，将x和y设置为150，宽和高设置为300。
5. 继续选中这个视图，点击Auto Layout工具栏中的Align按钮，勾选Horizontal Center in Container和Vertical Center in Container，将它们的值均设置为0，然后点击Add 2 Constraints。
6. 点击Auto Layout工具栏中的Pin按钮，勾选Width和Height，将它们的值设置为300，然后点击Add 2 Constraints。


接着，为这个视图创建一个名为viewForLayer的输出口。方法是选中这个视图，然后control－drag到ViewController.swift中。

然后将ViewController.swift的内容修改如下：
	
	import UIKit

	class ViewController: UIViewController {

    	@IBOutlet weak var viewForLayer: UIView!
    
    	var l :CALayer {
        	return viewForLayer.layer
    	}	
    
    	override func viewDidLoad() {
        	super.viewDidLoad()
        	setUpLayer()
    	}
    
	    func setUpLayer() -> () {
	        l.backgroundColor = UIColor.blueColor().CGColor
	        l.borderColor = UIColor.redColor().CGColor
	        l.borderWidth = 100.0
	        l.shadowOpacity = 0.7
	        l.shadowRadius = 10.0
	    }
	}

现在编译运行程序，你应该可以看到如下结果：
![Result](http://7xnyik.com1.z0.glb.clouddn.com/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202016-04-17%2021.12.27.png-picVer)

###CALayer的一些属性
CALayer拥有一些属性，你可以用这些属性来定义它的外观。在上面的例子中，我们已经

1. 将layer的背景色从默认值无背景色改为了蓝色。
2. 将layer的border宽度从0改为了100。
3. 将layer的默认颜色从黑色改为了红色。
4. 创建了一个阴影样式，并将它的透明度设置为了70%。然后，将阴影的半径从默认值3改为了10。

你还可以进行更多的设置。将下面两行代码加到setUpLayer()函数的尾部。

	l.contents = UIImage(named:"star")?.CGImage
	l.contentsGravity = kCAGravityCenter
	
将layer的contents属性设置为一张图片，然后将图片的显示方式设置为居中。

运行后的结果与上图基本相同，只是在蓝色正方形的中间出现了一个五角星。

###改变Layer的外观

给viewForLayer添加一个Tap和Pinch的Gesture Recognizer，然后在viewController.swift中创建两个动作，分别为tapGestureRecognized和pinchGestureRecogized。

将tapGestureRecognized(_:) 改为下面这样：

	@IBAction func tapGestureRecognized(sender:UITapGestureRecognizer) {
		l.shadowOpacity = l.shadowOpacity == 0.7?0.0:0.7;
	}
	
当用户tap这个视图时，视图的阴影透明度在0和0.7之间来回切换。

然后改写pinchGestureRecognized(_:)方法：
	
	@IBAction func pinchGestureRecognized(sender:UIPinchGestureRecognizer) {
		let offset: CGFloat = sender.scale < 1?5.0:-5.0
		let oldFrame = l.frame
		let oldOrigin = oldFrame.origin
		let newOrigin = CGPoint(x:oldOrigin.x + offset, y:oldOrigin.y + offset)
		let newSize = CGSize(width:oldFrame.width + ( offset * -2.0), height :oldFrame.height + (offset * -2.0))
		let newFrame.width >= 100.0 && newFrame.width <= 300.0 {
			l.borderWidth -= offset
			l.cornerRadius += (offset / 2.0)
			l.frame = newFrame
		}
	}
	
这样就可以使用手势缩放视图的边界大小了。
注意，使用layer的corner radius属性可以设置视图的圆角的半径。默认的圆角半径是0，表示的是一个直角矩形。如果把圆角半径设置为层宽度的半径，那么这个层就变成了圆形。

###CALayer Player
刚刚的那个小例子只为我们展现了CALayer的冰山一角，下面我们将用一个已经上架的App－CALayer Player来为你展现CALayer的更多细节。

####Example 1: CALayer
除了刚刚已经介绍过的外，CALayer还有如下特性：

* Layer可以有子Layer。
* Layer的属性改变通常伴随着一些动画效果。你可以自定义这些动画效果。
* Layer是轻量级的。
* Layer有很多有用的属性。

		//1
		let layer = CALayer()
		layer.frame = someView.bounds
		
		//2
		layer.contents = UIImage(named:"star")?.CGImage
		layer.contentsGravity = kCAGravityCenter
		
		//3
		layer.magnificationFilter = kCAFilterLinear
		layer.geometryFilpped = false
		
		//4
		layer.backgroundColor = UIColor(red:11/255.0,green:86/255.0,blue:14/255.0,alpha:1.0).CGColor
		layer.opacity = 1.0
		layer.hidden = false
		layer.masksToBounds = false
		
		//5
		layer.cornerRadius = 100.0
		layer.borderWidth = 12.0
		layer.borderColor = UIColor.whiteColor().CGColor
		
		//6
		layer.shadowOpacity = 0.75
		layer.shadowOffset = CGSize(width:0,height:3)
		layer.shadowRadius = 3.0
		layer.shouldRasterize = true
		someView.layer.addSublayer(layer)

上面的代码实际上做了下面几件事：
1. 创建了一个新的CALayer实例。
2. 将这个layer的内容设置为一幅图片并且居中。
3. 当对图片进行放大时，使用这个过滤器，它可以同时更改大小和位置。
4. 设置了layer的背景色、透明度和可见性。masksToBounds被设置为了false，因此当layer的尺寸小于图片尺寸时，图片将不会被剪裁。
5. 将layer圆角半径设置为了layer宽度的一半，因此边界变为了一个圆形。
6. 创建了一个阴影效果，并且将shouldRasterize设置为true。然后将layer设置为someView的layer的子layer

CALayer有两个能够提高性能的属性：`shouldRasterize`和`drawsAsynchronously`。

`shouldRasterize`默认为false，当它被设置为true时，因为layer的内容只需要渲染一次，因此可以提升性能。它很适合那些在屏幕周围但是不需要改变外观的对象。

`drawAsynchronously`从某种程度上可以说是shouldRasterize的反面。它的默认值也是false。当一个layer的内容必须被重复绘制时，将它设置为true可以提高性能。

你现在可以在Layer Player中对CALayer的许多属性进行设置了。动手试一试，看看那个图案会发生什么样的变化。

**注意**：Layer并不是响应链的一部分，因此它不能像视图一样直接对触摸或者手势做出响应。

####Example 2: CAScrollLayer
CAScrollLayer显示了一个可滚动的layer的一部分。它相当基础，不能对用户触摸动作做出反应，也不能检查是否到达了layer的边界。
UIScrollView没有使用CAScrollLayer来实现scroll，而是直接改变它自己的layer的边界。
你能够用CAScrollLayer做的就是更改它的滚动模式为水平或者垂直，或者是用代码来让它滚动到某个特殊的点或rect。

	//In ScrollingView.swift
	import UIKit

	class ScrollingView:UIView {
	//1
    	override class func layerClass() -> AnyClass {
        	return CAScrollLayer.self
       }
    }
    
    
	//In CAScrollLayerViewController.swift

	import UIKit

	class CAScrollLayerViewController: UIViewController {
	    @IBOutlet weak var scrollingView: ScrollingView!
	
	
		//2
	    var scrollingViewLayer:CAScrollLayer {
	        return scrollingView.layer as CAScrollLayer
	    }
	
	    override func viewDidLoad() {
	        super.viewDidLoad()
	        //3
	        scrollingViewLayer.scrollMode = kCAScrollBoth
	    }
	
	    @IBAction func tapRecognized(sender: UITapGestureRecognizer) {
	        
	        //4
	        var newPoint = CGPoint(x:250,y:250)
	        UIView.animateWithDuration(0.3, delay: 0, options: .CurveEaseInOut, animations: {
	            [unowned self] in
	            self.scrollingViewLayer.scrollToPoint(newPoint)
	        },completion:nil)
	    }
	}	
	
在上面的代码中：

1. 一个UIView的子类ScrollingView重载了layerClass()方法，使它返回了CAScrollLayer。当然也可以像Example 1一样，将创建的新layer变为原有view的layer的子layer。
2. 设置了一个计算属性
3. 将滚动模式设置为了水平和垂直。
4. 当识别出tap动作后，创建一个新point，让layer滚动到那个新点。需要注意的是scrollToPoint(_:)和scrollToRect(_:)并不带有动画，所以需要借助UIView的方法来实现动画效果。

使用CAScrollLayer有如下的注意事项：

1. 如果你希望使用代码来控制滚动，并且实现较为轻量级，则考虑使用CAScrollLayer。
2. 如果你希望用户可以用手指来滚动，那么使用UIScrollView。
3. 如果希望滚动一张非常大的图片，则可以使用CATiedLayer。

####Example 3: CATextLayer
CATextLayer能够对纯文本或者富文本进行简单而快速的渲染。与UILabel不同的是，CATextLayer不能有与之相关联的UIFont属性，但可以用CTFontRef或CGFontRef来对字体进行修改。

	
	//1
	let textLayer = CATextLayer()
	textLayer.frame = someView.bounds
	
	
	//2
	var string = ""
	for _ in 1...20 {
	    string += "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Fusce auctor arcu quis velit congue dictum. "
	}
	
	textLayer.string = string
	
	
	//3
	let fontName:CFStringRef = "Noteworthy-Light"
	textLayer.font = CTFontCreateWithName(fontName, fontSize,nil)
	
	//4
	textLayer.foregroundColor = UIColor.darkGrayColor().CGColor
	textLayer.wrapped = true
	textLayer.alignmentMode = kCAAlignmentLeft
	textLayer.contentsScale = UIScreen.mainScreen().scale
	someView.layer.addSublayer(textLayer)

对上面代码的解释；

1. 创建了一个CATextLayer的实例，将它的frame设置为someView的bounds。
2. 创建一个字符串。
3. 设置textLayer的字体。
4. 设置textLayer的前景色、是否换行、对齐方式、缩放比例。最后将该layer设置为someView的layer的子layer。

包括CATextLayer在内的所有layer，默认都是以1为缩放比例进行渲染的。当被绑定到视图上后，layers会针对当前屏幕大小自动将它们的contentsScale设置为合适的值。对于手工创建的layer来说，你必须明确地设置这个值，否则它们的缩放比例就会为1。

####Example 4: AVPlayerLayer
AVPlayerLayer为AVFoundation增加了一个不错的特性。它可以被用来播放媒体文件。下面是一个创建和使用AVPlayerLayer的示例。

	override func viewDidLoad() {
	    super.viewDidLoad()
	
		//1
	    let playerLayer = AVPlayerLayer()
	    playerLayer.frame = someView.bounds
	
		//2
	    let url = NSBundle.mainBundle().URLForResource("someVideo",withExtension:"m4v")
	    let player = AVPlayer(URL:url)
	
		//3
	    player.actionAtItemEnd = .None
	    playerLayer.player = player
	    someView.layer.addSubLayer(playerLayer)
	
		//4
	    NSNotificationCenter.defaultCenter().addObserver(self, selector:"playerDidReachEndNotificationHandler:",
	    name:"AVPlayerItemDidPlayToEndTimeNotification",object:player.currentItem)
	}

	deinit {
	    NSNotificationCenter.defaultCenter.removeObserver(self)
	}
	
	//5
	@IBAction func playButtonTapped(sender:UIButton) {
	    if  playButton.titleLabel?.text == "Play" {
	        player.play()
	        playButton.setTitle("Pause",forState:.Normal)
	    } else {
	        player.pause()
	        playButton.setTitle("Play",forState:.Normal)
	    }
	
	    updatePlayButtonTitle()
	    updateRateSegmentedControl()
	}
	
	//6
	func playerDidReachEndNotificationHandler(notification: NSNotification) {
	    let playerItem = notification.object as AVPlayerItem
	    playerItem.seekToTime(kCMTimeZero)
	}

上面的代码：

1. 创建了一个新的AVPlayerLayer实例，并且设置了它的frame。
2. 创建了一个AVPlayer实例。
3. 设置了当播放完毕时的动作。
4. 登记一个通知，并且设置事件的观察者。
5. 设置了按钮处理逻辑。
6. 设置了当收到第四步中发出的通知时的处理逻辑。

AVPlayerLayer有一些额外的的属性：

`videoGravity`设置了播放视频时的缩放行为。
`readyForDisplay`检查视频是否准备好播放。

AVPlayer也有一些特别的属性：一个值得注意的是`rate`，也就是播放速率。0代表着暂停，1代表以一倍速进行播放，2代表着以二倍速进行播放。这个属性和视频的播放与否是有关系的，当rate为0时，相当于调用了pause()函数，而当rate设置为1时，相当于调用了play()。当rate为负数时，意味着视频在逆序播放。

不过，在我们设置rate之前，我们需要先判断视频是否能够按照我们将设置的rate进行播放。这些判断函数是：

* canPlayFastForward(): if can play at rate >= 1
* canPlaySlowForward(): if can play at 0 <= 0 < 1
* canPlayReverse(): if can play at rate == -1
* canPlaySlowReverse(): if can play at -1 <= rate < 0
* canPlayFastReverse(): if can play at rate < -1

大部分视频都支持用不同的速度正向播放，但是很少有支持倒着播放的。

####Example 5: CAGradientLayer
下面是一个使用CAGradientLayer的例子：

    let gradientLayer = CAGradientLayer()
    gradientLayer.frame = someView.bounds
    gradientLayer.colors = [cgColorForRed(209.0,green:0.0,blue:0.0),
        cgColorForRed(255.0,green:102.0,blue:34.0),
        cgColorForRed(255.0,green:218.0,blue:33.0),
        cgColorForRed(51.0,green:221.0,blue:0.0),
        cgColorForRed(17.0,green:51.0,blue:204.0),
        cgColorForRed(34.0,green:0.0,blue:102.0),
        cgColorForRed(51.0,green:0.0,blue:68.0)]
        gradientLayer.startPoint = CGPoint(x:0,y:0)
        gradientLayer.endPoint = CGPoint(x:0,y:1)
        someView.layer.addSubLayer(gradientLayer)
    
    func cgColorForRed(red: CGFloat, green:CGFloat,blue :CGFloat) -> AnyObject {
        return UIColor(red:red/255.0,green:green/255.0,blue:blue/255.0,alpha:1.0).CGPoint as AnyObject
    }

CAGradientLayer能够轻松地混合多种颜色，因此很适合用来设置背景色。在进行配置的时候，你需要事先定义一个装有CGColors的数组，和指示混合起点和终点的`startPoint`、`endPoint`。
记住，`startPoint`和`endPoint`并不是实际的点。事实上，它们是定义在单位坐标系中的，因此在绘制的时候需要将它们映射到layer的bounds上。换句话说，x=1意味着这个点在layer的右边缘，y＝1意味着这个点在layer的底部。

CAGradientLayer有一个名为type的属性，但只有kCAGradientLayerAxial这一个选项，它可以使得layer的颜色在预先定义的颜色之间线性过渡。

CAGradientLayer还有一个locations属性，这个属性是一个和color相同大小的数组，它的值为0和1之间，并且每一个值指示着CAGradientLayer在什么相对位置处应该使用下一个颜色值。
如果不设置这个locations属性，那么默认CAGradientLayer在每一个等份点使用Color数组的下一个颜色。







