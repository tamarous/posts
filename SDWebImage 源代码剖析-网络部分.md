# SDWebImage 源代码剖析-网络部分

前面两篇博文中，简要对SDWebImage的[缓存部分](http://www.tamarous.com/2017/03/12/sdwebimage-cache-policy/)和[多线程部分](http://www.tamarous.com/2017/03/14/sdwebimage-multithread-policy/)进行了分析。建议在阅读本篇内容前先看下缓存策略那篇，以对SDWebImage的基本内容有所了解。在本篇中，我们将对它的网络策略进行分析。我们知道SDWebImage的主要功能就是从远程主机上下载图片，所以前面的几个策略都是为这一目的提供支持的，而网络策略的好坏将直接决定库的性能。不过SDWebImage以GitHub上接近20000的star数向我们证明了它不俗的实力，下面就让我们一起看看吧。

本文共分为两部分，第一部分是对NSURLSession的介绍，第二部分是对SDWebImage中网络部分的介绍。

## NSURLSession
NSURLSession是苹果在2013年的WWDC上推出的，目的是取代老迈的前任NSURLConnection。它包含了一系列组件，包括`NSURLRequest`、`NSURLCache`、`NSURLSessionConfiguration`、`NSURLSessionTask`等类。大名鼎鼎的AFNetworking以及本文要重点介绍的SDWebImage内部都大量使用NSURLSession来完成网络请求功能，因此了解NSURLSession是很有必要的。
如果没有什么额外需求，那么使用NSURLSession来发出一次网络请求是非常简单的：
    
    ```
    NSString *urlString = @"https://www.example.com";
    NSURL *url = [NSURL URLWithString:urlString];
    NSURLRequest *request = [NSURLRequest requestWithURL:url];
    [[NSURLSession sharedSession] dataTaskWithRequest:request completionHandler:^(NSData * _Nullable data, NSURLResponse * _Nullable response, NSError * _Nullable error) {
        if (data && !error) {
            NSString *response = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
            NSLog(@"response is %@",response);
        } else {
            NSLog(@"something wrong happened.");
        }
    }];
    ```
    
这里我们使用的是均为默认配置。如果我们想在进行网络请求时进行一些自定义，那么我们就需要使用`NSURLSessionConfiguration`类来配置我们的`NSURLSession`。

### NSURLSessionConfiguration
`NSURLSessionConfiguration`可以配置的东西很多，比如缓存策略、时延、连接要求、Cookies等等，几乎你能想到的东西都可以进行配置。熟练掌握这些可配置选项的用法和意义对于我们控制连接的行为重要。
`networkServiceType`：类型是`NSURLRequestNetworkServiceType`，指明了所处的网络环境，系统可能会针对性地使用不同的性能模式、电量模式；`allowsCellularAccess`，BOOL类型，指明连接是否可以通过蜂窝数据进行。某些APP如微博、知乎等提供只通过WiFi浏览的功能，其实就是利用这个属性来实现的；
`timeoutIntervalForRequest`，类型是NSTimeInterval，指明了所有task的超时时间。在这个时间之内，task会继续等待；当超过这个时间后，task会认为本次连接失败了，默认是60秒。

在初始化一个`NSURLSession`实例的时候我们传入一个`NSURLSessionConfiguration`实例，session会将这个configuration拷贝到自身中，一旦初始化完成了，对configuration进行修改并不会改变session的配置。如果我们要使用新的配置，那么我们只能重新生成一个NSURLSession实例。
`NSURLSessionConfiguration`提供了三个工厂方法：

1. defaultSessionConfiguration
这个是默认配置，它会将缓存存储在磁盘上、将凭证信息存储在用户的keychain以及使用全局的cookie信息。

2. ephemeralSessionConfiguration
这个配置不会将缓存、凭证信息、cookies等存储在磁盘上，而是存储在内存中，因此这些数据会被自动清除释放。因此当我们想要实现无痕浏览等功能的话就可以使用这个配置。

3. backgroundSessionConfigurationWithIdentifier:
这个配置允许应用进入后台时仍能进行下载上传工作。当使用这个配置时，session会把控制权交给系统，系统在另外的进程中进行数据传输。当应用停止并重启后，参数identifier可以用来恢复原来传输过程的长下文。不过呢，这个恢复也是有条件的，只有应用是由系统停止的，在停止后才能继续在后台传输；如果应用是由用户在多任务界面关掉的话，那么系统会取消掉所有后台传输。


使用`NSURLConfiguration`来初始化一个`NSURLSession`的代码如下：

```
NSURLSessionConfiguration *configuration = [NSURLSessionConfiguration defaultSessionConfiguration];
NSURLSession *session = [NSURLSession sessionWithConfiguration:configuration];
```

### NSURLSessionTask
`NSURLSession`字面上的意思是“一次会话”，在一次会话中可能会有多次请求，而每一次具体的请求是使用`NSURLSessionTask`来完成的。同一个NSURLSession可以创建多个task，并且这些task之间的cache 和cookie是共享的。NSURLSession 为我们提供了三种类型的task，分别是：

1. NSURLSessionDataTask：用于一般类型的数据传输，如客户端读取服务器返回的请求。

2. NSURLSessionDownloadTask：用于下载任务，它的内部针对大文件的网络请求做了更多处理。

3. NSURLSessionUploadTask：用于上传任务，它主要用于客户端向服务器发送数据，如用户上传头像、发送照片等场景

### Delegates
NSURLSession为我们提供了种类多样功能丰富的代理方法，以便我们能够自定义地处理网络请求过程中的各种情况。比如`NSURLSessionDataDelegate`中，
`- URLSession:dataTask:didReceiveResponse:completionHandler:`可以通知我们已经收到了服务器的响应；
`NSURLSessionDownloadDelegate`中，`URLSession:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:`可以告诉我们总共需要接收多少数据、现在已经接收多少数据，从而我们能够计算出下载的进度；`NSURLSessionDownloadDelegate`中的
`- URLSession:downloadTask:didFinishDownloadingToURL:`可以告诉我们下载已经完成并告诉我们下载的临时文件的地址。这里的介绍，只是抛砖引玉，更多详细的内容可以去查阅[URL Session Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/URLLoadingSystem/URLLoadingSystem.html)以及NSURLSession的相关文档。

### 小结
上面的内容简要介绍了`NSURLSession`这一系列API的使用方法，我们接下来会看到，`SDWebImage`的网络请求部分完全构建在`NSURLSession`之上，因此了解进而掌握`NSURLSession`这一系列框架是非常有必要的。

## SDWebImageDownloader

我们开始正式对`SDWebImage`中的网络部分进行分析。为了行文的流畅性以及清晰性，本篇内容与之前的博客内容会有所重复。
首先来看一下`SDWebImageDownloader`这个类。在SDWebImageDownloader.m中可以看到`SDWebImageDownloader`的一个扩展：

```
@interface SDWebImageDownloader () <NSURLSessionTaskDelegate, NSURLSessionDataDelegate>

// 一个NSOperationQueue，用来控制所有的SDWebDownloadOperation
@property (strong, nonatomic, nonnull) NSOperationQueue *downloadQueue;

// 上一个加入downloadQueue的operation
@property (weak, nonatomic, nullable) NSOperation *lastAddedOperation;

@property (assign, nonatomic, nullable) Class operationClass;

// 一个键值对，存储当前正在下载中的图片URL和对应的downloadOperation
@property (strong, nonatomic, nonnull) NSMutableDictionary<NSURL *, SDWebImageDownloaderOperation *> *URLOperations;

// 存储HTTP头部的词典
@property (strong, nonatomic, nullable) SDHTTPHeadersMutableDictionary *HTTPHeaders;
// This queue is used to serialize the handling of the network responses of all the download operation in a single queue
@property (SDDispatchQueueSetterSementics, nonatomic, nullable) dispatch_queue_t barrierQueue;

// The session in which data tasks will run
// 用来进行网络下载的session
@property (strong, nonatomic) NSURLSession *session;

@end
```

这个类实现了`NSURLSessionTaskDelegate`和`NSURLSessionDataDelegate`两个协议，显然它就是实际进行网络下载的类了。

### 初始化SDWebImageDownloader
`SDWebImageDownloader`有一个便捷构造器和一个指定构造器：

```
- (nonnull instancetype)init {
    // 用默认的NSURLSessionConfiguration来调用指定构造器
    return [self initWithSessionConfiguration:[NSURLSessionConfiguration defaultSessionConfiguration]];
}

- (nonnull instancetype)initWithSessionConfiguration:(nullable NSURLSessionConfiguration *)sessionConfiguration {
    if ((self = [super init])) {
        _operationClass = [SDWebImageDownloaderOperation class];
        _shouldDecompressImages = YES;
        
        // 默认是先入先出顺序，内部实现应该是一个队列
        _executionOrder = SDWebImageDownloaderFIFOExecutionOrder;
        
        // 用于下载的NSOperationQueue
        _downloadQueue = [NSOperationQueue new];
        
        // 该队列允许的最大并发操作数为6，也就是同时可以有6个Operation在同时进行
        _downloadQueue.maxConcurrentOperationCount = 6;
        _downloadQueue.name = @"com.hackemist.SDWebImageDownloader";
        
        // 用于将图片的URL和下载该图片的Operation对应起来
        _URLOperations = [NSMutableDictionary new];
#ifdef SD_WEBP
        _HTTPHeaders = [@{@"Accept": @"image/webp,image/*;q=0.8"} mutableCopy];
#else
        _HTTPHeaders = [@{@"Accept": @"image/*;q=0.8"} mutableCopy];
#endif

        // 创建一个dispatch_queue
        _barrierQueue = dispatch_queue_create("com.hackemist.SDWebImageDownloaderBarrierQueue", DISPATCH_QUEUE_CONCURRENT);
        _downloadTimeout = 15.0;

        [self createNewSessionWithConfiguration:sessionConfiguration];
    }
    return self;
}

- (void)createNewSessionWithConfiguration:(NSURLSessionConfiguration *)sessionConfiguration {
    [self cancelAllDownloads];

    // 如果已经有一个session了，那么就先让这个session无效并且取消掉它
    if (self.session) {
        [self.session invalidateAndCancel];
    }


    sessionConfiguration.timeoutIntervalForRequest = self.downloadTimeout;

    /**
     *  Create the session for this task
     *  We send nil as delegate queue so that the session creates a serial operation queue for performing all delegate
     *  method calls and completion handler calls.
     */
     
     // 将session 的delegate设置为SDWebImageDownloader自身
    self.session = [NSURLSession sessionWithConfiguration:sessionConfiguration
                                                 delegate:self
                                            delegateQueue:nil];
}
```

### 下载过程

```
- (nullable SDWebImageDownloadToken *)downloadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageDownloaderOptions)options
                                                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                                 completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock {
    __weak SDWebImageDownloader *wself = self;

    return [self addProgressCallback:progressBlock completedBlock:completedBlock forURL:url createCallback:^SDWebImageDownloaderOperation *{
        //
        // 
    }                                                 
}
```
该方法其实是调用了下面的这个方法，而//这部分的代码是createCallback这个block块的内容。我们先看一下下面这个方法里做了什么。

```
- (nullable SDWebImageDownloadToken *)addProgressCallback:(SDWebImageDownloaderProgressBlock)progressBlock
                                           completedBlock:(SDWebImageDownloaderCompletedBlock)completedBlock
                                                   forURL:(nullable NSURL *)url
                                           createCallback:(SDWebImageDownloaderOperation *(^)(void))createCallback {
    // The URL will be used as the key to the callbacks dictionary so it cannot be nil. If it is nil immediately call the completed block with no image or data.
    if (url == nil) {
        if (completedBlock != nil) {
            completedBlock(nil, nil, nil, NO);
        }
        return nil;
    }

    // 因为这个变量会在下面的block里面被修改，因此我们要将它声明为_block的
    __block SDWebImageDownloadToken *token = nil;

    // dispatch_barrier_sync：执行block，并且等待它的执行结果
    dispatch_barrier_sync(self.barrierQueue, ^{
    
        // 取出该URL对应的operation
        SDWebImageDownloaderOperation *operation = self.URLOperations[url];
        if (!operation) {
            // 如果该operation在词典中不存在的话，就在词典中新加一个条目
            operation = createCallback();
            self.URLOperations[url] = operation;

            __weak SDWebImageDownloaderOperation *woperation = operation;
            
            // 当该operation执行完的时候，调用completionBlock来从
            // 词典中把这个url对应的operation删掉
            operation.completionBlock = ^{
				dispatch_barrier_sync(self.barrierQueue, ^{
					SDWebImageDownloaderOperation *soperation = woperation;
					if (!soperation) return;
					if (self.URLOperations[url] == soperation) {
						[self.URLOperations removeObjectForKey:url];
					};
				});
            };
        }
        // 设置用于取消该operation的token
        id downloadOperationCancelToken = [operation addHandlersForProgress:progressBlock completed:completedBlock];

        token = [SDWebImageDownloadToken new];
        token.url = url;
        token.downloadOperationCancelToken = downloadOperationCancelToken;
    });

    return token;
}
```
从这个方法中我们可以看出`self.URLOperation`这个属性，是以URL为key以operation为value的一个键值对，而且由于operation会在自己执行完毕时将自己移除出键值对，因此`self.URLOperation`中的operation都处在尚未执行完毕的状态。
我们再回过头来看看在`- (nullable SDWebImageDownloadToken *)downloadImageWithURL:
options:progress:completed:completedBlock`方法中createCallback的内容。


```
- (nullable SDWebImageDownloadToken *)downloadImageWithURL:(nullable NSURL *)url
                                                   options:(SDWebImageDownloaderOptions)options
                                                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                                 completed:(nullable SDWebImageDownloaderCompletedBlock)completedBlock {
    __weak SDWebImageDownloader *wself = self;

    return [self addProgressCallback:progressBlock completedBlock:completedBlock forURL:url createCallback:^SDWebImageDownloaderOperation *{
        __strong __typeof (wself) sself = wself;
        NSTimeInterval timeoutInterval = sself.downloadTimeout;
        if (timeoutInterval == 0.0) {
            timeoutInterval = 15.0;
        }

        // In order to prevent from potential duplicate caching (NSURLCache + SDImageCache) we disable the cache for image requests if told otherwise
        
        // 1.
        NSURLRequestCachePolicy cachePolicy = options & SDWebImageDownloaderUseNSURLCache ? NSURLRequestUseProtocolCachePolicy : NSURLRequestReloadIgnoringLocalCacheData;
        NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:url
                                                                    cachePolicy:cachePolicy
                                                                timeoutInterval:timeoutInterval];
        
        request.HTTPShouldHandleCookies = (options & SDWebImageDownloaderHandleCookies);
        request.HTTPShouldUsePipelining = YES;
        if (sself.headersFilter) {
            request.allHTTPHeaderFields = sself.headersFilter(url, [sself.HTTPHeaders copy]);
        }
        else {
            request.allHTTPHeaderFields = sself.HTTPHeaders;
        }
        
        // 2.
        SDWebImageDownloaderOperation *operation = [[sself.operationClass alloc] initWithRequest:request inSession:sself.session options:options];
        operation.shouldDecompressImages = sself.shouldDecompressImages;
        
        if (sself.urlCredential) {
            operation.credential = sself.urlCredential;
        } else if (sself.username && sself.password) {
            operation.credential = [NSURLCredential credentialWithUser:sself.username password:sself.password persistence:NSURLCredentialPersistenceForSession];
        }
        
        if (options & SDWebImageDownloaderHighPriority) {
            operation.queuePriority = NSOperationQueuePriorityHigh;
        } else if (options & SDWebImageDownloaderLowPriority) {
            operation.queuePriority = NSOperationQueuePriorityLow;
        }

        // 3.
        [sself.downloadQueue addOperation:operation];
        if (sself.executionOrder == SDWebImageDownloaderLIFOExecutionOrder) {
            // Emulate LIFO execution order by systematically adding new operations as last operation's dependency
            [sself.lastAddedOperation addDependency:operation];
            sself.lastAddedOperation = operation;
        }

        return operation;
    }                                                 
}
```

1. 使用`SDWebImageDownloaderOptions`来配置一个NSURLRequest对象
2. 创建一个`SDWebImageDownloaderOperation`对象并进行配置
3. 将第二步中创建的operation加入到self.downloadQueue中。如果我们使用的执行策略是后入先出，也就是说上一个operation会在当前的这个operation执行完才执行，因此我们把上一个operation的依赖设置为当前operation。

### 协议实现
上面提到过`SDWebImageDownloader`这个类实现了`NSURLSessionTaskDelegate`和`NSURLSessionDataDelegate`两个协议。但是当我们去查看实现时我们发现这些协议方法内都是调用了`SDWebImageDownloaderOperation`对象的同名方法。

```
#pragma mark NSURLSessionDataDelegate

- (void)URLSession:(NSURLSession *)session
          dataTask:(NSURLSessionDataTask *)dataTask
didReceiveResponse:(NSURLResponse *)response
 completionHandler:(void (^)(NSURLSessionResponseDisposition disposition))completionHandler {

    // Identify the operation that runs this task and pass it the delegate method
    SDWebImageDownloaderOperation *dataOperation = [self operationWithTask:dataTask];

    [dataOperation URLSession:session dataTask:dataTask didReceiveResponse:response completionHandler:completionHandler];
}
```

### 小结
从代码中我们发现，`SDWebImageDownloader`完成的核心功能无非就是管理着`self.URLOperations`键值对，而协议方法其实都是由`SDWebImageDownloaderOperation`来完成的。
## SDWebImageDownloaderOperation

这个类比较关键的属性有如下几个：

```
// 该operation中的dataTask所接受的NSURLRequest对象
@property (strong, nonatomic, readonly, nullable) NSURLRequest *request;

// 一个NSURLSessionTask对象，用来执行下载任务
@property (strong, nonatomic, readonly, nullable) NSURLSessionTask *dataTask;

// 从服务器上返回的响应
@property (strong, nonatomic, nullable) NSURLResponse *response;

// 一个用来存放回调函数的数组
typedef NSMutableDictionary<NSString *, id> SDCallbacksDictionary;
@property (strong, nonatomic, nonnull) NSMutableArray<SDCallbacksDictionary *> *callbackBlocks;

```
因为该类继承了NSOperation，因此也实现了`start`、`cancel`等方法。

```
// 为了简便而略去了一些不重要的部分。
- (void)start {
    @synchronized (self) {
        if (self.isCancelled) {
            self.finished = YES;
            [self reset];
            return;
        }

        if (self.options & SDWebImageDownloaderIgnoreCachedResponse) {
            // Grab the cached data for later check
            NSCachedURLResponse *cachedResponse = [[NSURLCache sharedURLCache] cachedResponseForRequest:self.request];
            if (cachedResponse) {
                self.cachedData = cachedResponse.data;
            }
        }
        
        NSURLSession *session = self.unownedSession;
        if (!self.unownedSession) {
            NSURLSessionConfiguration *sessionConfig = [NSURLSessionConfiguration defaultSessionConfiguration];
            sessionConfig.timeoutIntervalForRequest = 15;
            
            // 使用默认配置来创建一个NSURLSession
            self.ownedSession = [NSURLSession sessionWithConfiguration:sessionConfig
                                                              delegate:self
                                                         delegateQueue:nil];
            session = self.ownedSession;
        }
        
        // 创建一个NSURLSessionDataTask
        self.dataTask = [session dataTaskWithRequest:self.request];
        self.executing = YES;
    }
    
    // 开始执行下载图片的任务
    [self.dataTask resume];

    if (self.dataTask) {
        for (SDWebImageDownloaderProgressBlock progressBlock in [self callbacksForKey:kProgressCallbackKey]) {
            progressBlock(0, NSURLResponseUnknownLength, self.request.URL);
        }
        __weak typeof(self) weakSelf = self;
        
        // 通知开始下载的事件
        dispatch_async(dispatch_get_main_queue(), ^{
            [[NSNotificationCenter defaultCenter] postNotificationName:SDWebImageDownloadStartNotification object:weakSelf];
        });
    } else {
        [self callCompletionBlocksWithError:[NSError errorWithDomain:NSURLErrorDomain code:0 userInfo:@{NSLocalizedDescriptionKey : @"Connection can't be initialized"}]];
    }
}
```

### NSURLSessionDataDelegate
`NSURLSessionDataDelegate`共定义了五个方法，`SDWebImageDownloadOperation`实现了其中的三个：

```

// 收到了服务器的响应
- (void)URLSession:(NSURLSession *)session
          dataTask:(NSURLSessionDataTask *)dataTask
didReceiveResponse:(NSURLResponse *)response
 completionHandler:(void (^)(NSURLSessionResponseDisposition disposition))completionHandler {
    
    if (![response respondsToSelector:@selector(statusCode)] || (((NSHTTPURLResponse *)response).statusCode < 400 && ((NSHTTPURLResponse *)response).statusCode != 304)) {
    
        // 从响应中获得内容长度
        NSInteger expected = (NSInteger)response.expectedContentLength;
        expected = expected > 0 ? expected : 0;
        self.expectedSize = expected;
        for (SDWebImageDownloaderProgressBlock progressBlock in [self callbacksForKey:kProgressCallbackKey]) {
            progressBlock(0, expected, self.request.URL);
        }
        
        // 根据内容长度开辟一块内存空间用来存储图像数据
        self.imageData = [[NSMutableData alloc] initWithCapacity:expected];
        self.response = response;
        __weak typeof(self) weakSelf = self;
        dispatch_async(dispatch_get_main_queue(), ^{
            [[NSNotificationCenter defaultCenter] postNotificationName:SDWebImageDownloadReceiveResponseNotification object:weakSelf];
        });
    } else {
        NSUInteger code = ((NSHTTPURLResponse *)response).statusCode;
        
        //'304 Not Modified' 表示自上次请求后，对应的内容没有修改，因此可以使用已经缓存的数据
        if (code == 304) {
            [self cancelInternal];
        } else {
            [self.dataTask cancel];
        }
        __weak typeof(self) weakSelf = self;
        dispatch_async(dispatch_get_main_queue(), ^{
            [[NSNotificationCenter defaultCenter] postNotificationName:SDWebImageDownloadStopNotification object:weakSelf];
        });
        
        [self callCompletionBlocksWithError:[NSError errorWithDomain:NSURLErrorDomain code:((NSHTTPURLResponse *)response).statusCode userInfo:nil]];

        [self done];
    }
    
    if (completionHandler) {
        completionHandler(NSURLSessionResponseAllow);
    }
}

// 从服务器上获得了数据
- (void)URLSession:(NSURLSession *)session dataTask:(NSURLSessionDataTask *)dataTask didReceiveData:(NSData *)data {

    // 将下载到的数据追加到前面开辟的图像数据中
    [self.imageData appendData:data];

    if ((self.options & SDWebImageDownloaderProgressiveDownload) && self.expectedSize > 0) {
        // Get the image data
        NSData *imageData = [self.imageData copy];
        // Get the total bytes downloaded
        const NSInteger totalSize = imageData.length;
        // Get the finish status
        BOOL finished = (totalSize >= self.expectedSize);
        
        if (!self.progressiveCoder) {
            // We need to create a new instance for progressive decoding to avoid conflicts
            for (id<SDWebImageCoder>coder in [SDWebImageCodersManager sharedInstance].coders) {
                if ([coder conformsToProtocol:@protocol(SDWebImageProgressiveCoder)] &&
                    [((id<SDWebImageProgressiveCoder>)coder) canIncrementallyDecodeFromData:imageData]) {
                    self.progressiveCoder = [[[coder class] alloc] init];
                    break;
                }
            }
        }
        
        UIImage *image = [self.progressiveCoder incrementallyDecodedImageWithData:imageData finished:finished];
        if (image) {
            NSString *key = [[SDWebImageManager sharedManager] cacheKeyForURL:self.request.URL];
            image = [self scaledImageForKey:key image:image];
            if (self.shouldDecompressImages) {
                image = [[SDWebImageCodersManager sharedInstance] decompressedImageWithImage:image data:&data options:@{SDWebImageCoderScaleDownLargeImagesKey: @(NO)}];
            }
            
            [self callCompletionBlocksWithImage:image imageData:nil error:nil finished:NO];
        }
    }

    for (SDWebImageDownloaderProgressBlock progressBlock in [self callbacksForKey:kProgressCallbackKey]) {
        progressBlock(self.imageData.length, self.expectedSize, self.request.URL);
    }
}

// 询问delegate是否应该缓存响应内容
- (void)URLSession:(NSURLSession *)session
          dataTask:(NSURLSessionDataTask *)dataTask
 willCacheResponse:(NSCachedURLResponse *)proposedResponse
 completionHandler:(void (^)(NSCachedURLResponse *cachedResponse))completionHandler {
    
    NSCachedURLResponse *cachedResponse = proposedResponse;

    if (self.request.cachePolicy == NSURLRequestReloadIgnoringLocalCacheData) {
        // Prevents caching of responses
        cachedResponse = nil;
    }
    if (completionHandler) {
        completionHandler(cachedResponse);
    }
}
```

#### NSURLSessionTaskDelegate
`NSURLSessionTaskDelegate`中定义了八个方法，而`SDWebImageDownloadOperation`实现了其中两个：

```
// 告诉delegate下载完成了，如果有错误，那么error不为空
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didCompleteWithError:(NSError *)error {

    @synchronized(self) {
        self.dataTask = nil;
        __weak typeof(self) weakSelf = self;
        
        // 在主线程中发送下载停止的通知，如果error为nil，再发送下载完成的通知
        dispatch_async(dispatch_get_main_queue(), ^{
            [[NSNotificationCenter defaultCenter] postNotificationName:SDWebImageDownloadStopNotification object:weakSelf];
            if (!error) {
                [[NSNotificationCenter defaultCenter] postNotificationName:SDWebImageDownloadFinishNotification object:weakSelf];
            }
        });
    }
    
    if (error) {
        [self callCompletionBlocksWithError:error];
    } else {
        if ([self callbacksForKey:kCompletedCallbackKey].count > 0) {
            /**
             *  If you specified to use `NSURLCache`, then the response you get here is what you need.
             */
            NSData *imageData = [self.imageData copy];
            if (imageData) {
                /**  if you specified to only use cached data via `SDWebImageDownloaderIgnoreCachedResponse`,
                 *  then we should check if the cached data is equal to image data
                 */
                if (self.options & SDWebImageDownloaderIgnoreCachedResponse && [self.cachedData isEqualToData:imageData]) {
                    // call completion block with nil
                    [self callCompletionBlocksWithImage:nil imageData:nil error:nil finished:YES];
                } else {
                    UIImage *image = [[SDWebImageCodersManager sharedInstance] decodedImageWithData:imageData];
                    NSString *key = [[SDWebImageManager sharedManager] cacheKeyForURL:self.request.URL];
                    image = [self scaledImageForKey:key image:image];
                    
                    BOOL shouldDecode = YES;
                    // Do not force decoding animated GIFs and WebPs
                    if (image.images) {
                        shouldDecode = NO;
                    } else {
#ifdef SD_WEBP
                        SDImageFormat imageFormat = [NSData sd_imageFormatForImageData:imageData];
                        if (imageFormat == SDImageFormatWebP) {
                            shouldDecode = NO;
                        }
#endif
                    }
                    
                    if (shouldDecode) {
                        if (self.shouldDecompressImages) {
                            BOOL shouldScaleDown = self.options & SDWebImageDownloaderScaleDownLargeImages;
                            image = [[SDWebImageCodersManager sharedInstance] decompressedImageWithImage:image data:&imageData options:@{SDWebImageCoderScaleDownLargeImagesKey: @(shouldScaleDown)}];
                        }
                    }
                    if (CGSizeEqualToSize(image.size, CGSizeZero)) {
                        [self callCompletionBlocksWithError:[NSError errorWithDomain:SDWebImageErrorDomain code:0 userInfo:@{NSLocalizedDescriptionKey : @"Downloaded image has 0 pixels"}]];
                    } else {
                        [self callCompletionBlocksWithImage:image imageData:imageData error:nil finished:YES];
                    }
                }
            } else {
                [self callCompletionBlocksWithError:[NSError errorWithDomain:SDWebImageErrorDomain code:0 userInfo:@{NSLocalizedDescriptionKey : @"Image data is nil"}]];
            }
        }
    }
    [self done];
}

// 服务器要求进行验证，询问delegate如何处理
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential *credential))completionHandler {
    
    NSURLSessionAuthChallengeDisposition disposition = NSURLSessionAuthChallengePerformDefaultHandling;
    __block NSURLCredential *credential = nil;
    
    if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust]) {
        if (!(self.options & SDWebImageDownloaderAllowInvalidSSLCertificates)) {
            disposition = NSURLSessionAuthChallengePerformDefaultHandling;
        } else {
            credential = [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
            disposition = NSURLSessionAuthChallengeUseCredential;
        }
    } else {
        if (challenge.previousFailureCount == 0) {
            if (self.credential) {
                credential = self.credential;
                disposition = NSURLSessionAuthChallengeUseCredential;
            } else {
                disposition = NSURLSessionAuthChallengeCancelAuthenticationChallenge;
            }
        } else {
            disposition = NSURLSessionAuthChallengeCancelAuthenticationChallenge;
        }
    }
    
    if (completionHandler) {
        completionHandler(disposition, credential);
    }
}
```

## 总结
本文简单介绍了`NSURLSession`的使用，并分析了它在`SDWebImage`中的应用。至此，对`SDWebImage`这一框架的分析将告一段落。在分析`SDWebImage`源码的过程中，我曾数次被这一框架的优点打动：接口简洁易用，结构优美逻辑强，代码清晰易懂，注释详细丰富，值得再三学习品味。












