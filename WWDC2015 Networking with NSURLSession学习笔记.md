# WWDC2015 Networking with NSURLSession学习笔记

这个 Session 是我观看的第一个 WWDC Seesion，在线地址在[这里](https://developer.apple.com/videos/play/wwdc2015/711/)。事实证明观看 WWDC Session 的确是一种很好的学习方式，在短短的四十分钟内让我了解到了很多新的知识。不过由于知识点比较密集，因此要想完全掌握这些知识点，还需要之后多花点时间仔细查阅和学习相关文档。
这个Session 总共分为四个部分，分别是`App Transport Security`，`New protocol support in NSURLSession`，`NSURLSession on watchOS`以及`NSURLSession API changes`，其中我感觉比较有价值的是第二和第三部分~
## App Transport Security
从 iOS 9开始，Apple 要求所有应用都要使用 HTTPS 来和服务器进行通信。这样就要求应用的服务器必须支持 HTTPS 了，那么世界上那么多台服务器，有的好好地跑了那么多年，难不成为了支持 iOS 9，还要特意升级吗？苹果也意识到了这个问题，因此并没有非常强硬，而只是建议开发者都使用 HTTPS。如果实在没有办法，苹果也提出了一些变通之道。

在应用的 Info.plist 之中，可以添加一个名为 NSAppTransportSecurity 的键值对，然后将例外的域名加入到 Key 为 NSExceptionDomains 的 dictionary 中，或者是将 NSAllowsArbitraryLoads 设置为 YES 来允许所有流量都通过 HTTP。

![App Transport Security](http://7xnyik.com1.z0.glb.clouddn.com/AppTransportSecurity.jpg)

## New protocol support in NSURLSession
这部分主要介绍 NSURLSession 对 HTTP/2 的支持。目前最为广泛使用的 HTTP 协议仍是 HTTP/1.1，那么 HTTP/2和它之间有何异同呢？
从下面这个表格中我们可以略知一二：

|HTTP/1,1|HTTP/2|
|:---:|:---:|
|Multiple TCP Connections to a host|Only one TCP connection|
|Head-of-line blocking|Fully multiplexed|
|FIFO restrictions|Requests have priorities|
|Textual protocol overhead|Binary|

对于 HTTP/1.1和 HTTP/2更加深入的比较可以看这篇[博客](http://www.jianshu.com/p/52d86558ca57)。

## NSURLSession on watchOS
这一部分主要介绍了在watchOS 上使用NSURLSession的一些最佳实践，由于暂时用不上因此先不看了。

## NSURLSession API changes
### 废弃 NSURLConnection
在 iOS 9和 OS X 10.11中，苹果将 NSURLConnection标记为已过时，这意味着后续将不会向这个 API 加入任何新的功能了， 取代它的则是 NSURLSession。

发出一次异步请求，使用 NSURLConnection 的代码如下：

```
let queue: NSOperationQueue = ...
let url: NSURL(string: "https://www.example.com")
let request = NSURLRequest(URL:url)

NSURLConnection.sendAsynchronousRequest(request, queue: queue) {
    (response: NSURLResponse?, data: NSData?, error: NSError?) in 
    ...
}
```
而使用 NSURLSession 的代码则如下：

```
let sessionConfig = NSURLSessionConfiguration.defaultSessionConfiguration()
let session = NSURLSession(configuration: sessionConfig)
let url = NSURL(string:"https://www.example.com")!
let task = session.dataTaskWithURL(url) {
    (data:NSData?, response: NSURLResponse?, error: NSError?) in 
    ...
}
task?.resume()
```
可以看出使用 NSURLSession 比 NSURLConnection 要多了一步，也就是需要创建一个 NSURLSessionConfiguration 对象来对 session 进行配置，不过也正是由于这个 configuration 的存在，使得 NSURLSession 相比 NSURLConnection 显得更加灵活方便。

### 在应用和扩展之间共享 Cookies
默认情况下应用和扩展拥有各自不同的Cookies，但是有时候需要在这两者之间共享一些 Cookies，那么新提供的NSHttpCookieStroage API就可以完成这一需求。

```
let identifier = "group.mycompany.mygroupname"
let cookieStorage = NSHttpCookieStorage.sharedCookieStorageForGroupContainerIdentifier(identifier:identifier)
let config = NSURLSessionConfiguration.defaultSessionConfiguration()
config.HTTPCookieStorage = cookieStorage
let session = NSURLSession(configuration: config)
```







