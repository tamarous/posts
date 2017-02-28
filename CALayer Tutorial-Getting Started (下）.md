在前一篇文章中，我们介绍了CALayer的一部分特性。在这篇文章中，我们将继续对CALayer进行介绍。

####Example 6: CAReplicatorLayer
CAReplicatorLayer可以将一个layer复制指定的次数，因此可以用来实现一些酷炫的效果。
下面是一个例子：

    //1
    let replicatorLayer = CAReplicatorLayer()
    replicatorLayer.frame = someView.bounds
    
    
    //2
    replicatorLayer.instanceCount = 30
    replicatorLayer.instanceDelay = CFTimeInterval(1 / 30.0)
    replicatorLayer.preservesDepth = false
    replicatorLayer.instanceColor = UIColor.whiteColor().CGColor
    
    //3
    replicatorLayer.instanceRedOffset = 0.0
    replicatorLayer.instanceGreenOffset = -0.5
    replicatorLayer.instanceBlueOffset = -0.5
    replicatorLayer.instanceAlphaOffset = 0.0
    
    //4
    let angle = Float(M_PI * 2.0) / 30
    replicatorLayer.instanceTransform = CATransform3DMakeRotation(CGFloat(angle),0.0,0.0,1.0)
    someView.layer.addSublayer(replicatorLayer)
    
    //5
    let instanceLayer = CALayer()
    let layerWidth :CGFloat = 10.0
    let midX = CGRectGetMidX(someView.bounds) - layerWidth/2.0
    instanceLayer.frame = CGRect(x:midX, y:0.0, width:layerWidth, height: layerWidth * 3.0)
    instanceLayer.backgroundColor = UIColor.whiteColor().CGColor
    replicatorLayer.repeatCount = Float(Int.max)
    
    //6
    let fadeAnimation = CABasicAnimation(keyPath: "opacity")
    fadeAnimation.fromValue = 1.0
    fadeAnimation.toValue = 0.0
    fadeAnimation.duration = 1
    fadeAnimation.repeatCount = Float(Int.max)
    
    //7
    instanceLayer.opacity = 0.0
    instanceLayer.addAnimation(fadeAnimation,forKey:"FadeAnimation")
    
上面的代码：

1. 创建了一个CAReplicatorLayer的实例，设置了它的frame。
2. 用instanceCount设置了拷贝的次数，用instanceDelay设置了拷贝的绘制时延。