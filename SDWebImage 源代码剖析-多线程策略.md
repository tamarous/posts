# SDWebImage 源代码剖析-多线程策略
前一篇[文章](http://www.tamarous.com/2017/03/12/sdwebimage-code-review/)从缓存策略的角度分析了`SDWebImage` 的部分代码，下面从多线程的角度对它的其他模块进行分析。

苹果的多线程解决方案有三种：
* NSThread
* GCD
* NSOperation

在实际开发中，我们往往只使用GCD 和NSOperation。
关于应该如何在GCD 和NSOperation 中进行选择，stackoverflow 上已经有很多详细和深入的[讨论](http://stackoverflow.com/a/10375616/6552680)。本文中，我们只介绍NSOperation 。


## NSOperation
NSOperation 定义在Foundation 框架中，表示一个用来完成某项工作的操作。它是一个抽象类，因此在实际使用时，必须让一个类继承它，然后实现某些方法。Foundation 为我们事先定义好了两个子类，分别是`NSInvocationOperation` 和`NSBlockOperation`，这两个类可以直接使用，但是它们的适用场景是不同的：


类 | 描述
---- | ------
NSInvocationOperation | A class you use as-is to create an operation object based on an object and selector from your application. You can use this class in cases where you have an existing method that already performs the needed task.
NSBlockOperation | A class you use as-is to execute one or more block objects concurrently. Because it can execute more than one block, a block operation object operates using a group semantic; only when all of the associated blocks have finished executing is the operation itself considered finished.

### 实现自定义的NSOperation 子类
如果说上面提到的两个子类不能满足我们的需求，我们还可以自己去实现一个NSOperation 的子类。
如果你的这个子类是非并行的，那很好办，只要实现下面两个方法，加上对取消操作的响应即可；如果子类是并行的，那稍微复杂一些，还需要重载NSOperation 中的某些方法。
我们先看下并行和串行子类都必须实现的几个方法。

####每个operation 对象都应该实现的方法

* 一个自定义`initialization` 方法
* `main` 方法

`initialization` 用于对operation 对象进行初始化设置，`main`用于执行具体的任务。

其他的方法则可以根据需要来实现，比如说：

* 用于在`main` 中调用的辅助方法
* 用于设置和获取operation 对象的数据的`accessor`方法
* 用于序列化和反序列化的NSCoding 协议中的某些方法


下面是一个NSOperation 子类可能的样子：


```
@interface MyNonConcurrentOperation : NSOperation
@property id (strong) myData;
-(id)initWithData:(id)data;
@end
 
@implementation MyNonConcurrentOperation
- (id)initWithData:(id)data {
   if (self = [super init])
      myData = data;
   return self;
}
 
-(void)main {
   @try {
      // Do some work on myData and report the results.
   }
   @catch(...) {
      // Do not rethrow exceptions.
   }
}
@end
```

#### 对取消操作做出响应
注意，上面的那个例子没有实现对取消操作的响应。而事实上，对取消事件作出响应是operation 对象的一个重要工作。当一个operation 对象被执行后，它会一直运行，直到任务完成或者是操作被取消，而取消操作甚至可能发生在任务开始执行之前。当操作被取消的时候，operation 对象需要回收已经被分配的资源，并且优雅地退出。

为了能对取消操作作出反应，在代码中应该周期性地调用operation 对象的`isCancelled` 方法。更进一步地说，应该是在如下几个场景处调用此方法：

* 当进行任何实际性的工作时。
* 在每次循环中至少调用一次，如果每次循环的时间很久，则可以在一次循环中多次调用。这个方法本身的开销很小，因此不必担心多次调用会带来性能上的下降。
* 当取消操作很可能发生的时候。


```
- (void)main {
   @try {
      BOOL isDone = NO;
 
      while (![self isCancelled] && !isDone) {
          // Do some work and set isDone to YES when finished
      }
   }
   @catch(...) {
      // Do not rethrow exceptions.
   }
}

```

#### 并行operation 对象的配置
> Operation objects execute in a synchronous manner by default—that is, they perform their task in the thread that calls their start method. 


下面这张表列出了并行operation 对象所需要重载的一些方法。

方法 | 类型 | 描述
--- | ---- | ---
start | 必须实现 | 通常在这个方法中来设置并行任务的执行环境，如具体的线程等。这个方法是operation 对象的起始点。在这个方法中不可以调用super 的方法。
main | 可选的 | 在这个方法中实现具体要完成的任务。虽然实现也可以放在start 方法中来做，但是放在main 方法中可以使得设置和执行分离，使得代码的逻辑更加清晰。
isExecuting, isFinished | 必须实现 | 这两个方法用来向外部对象报告operation 对象的运行状态。 这两个方法的实现必须是线程安全的。
isConcurrent | 必须实现 | 为了确认一个operation 对象是否是并行的，需要重载这个方法，让它返回YES。


有了以上知识作铺垫，我们来看下一个并行operation 对象应该怎么完整地实现：

```
@interface MyOperation : NSOperation {
    BOOL        executing;
    BOOL        finished;
}
- (void)completeOperation;
@end
 
@implementation MyOperation
- (id)init {
    self = [super init];
    if (self) {
        executing = NO;
        finished = NO;
    }
    return self;
}
 
- (void)start {
   // Always check for cancellation before launching the task.
   if ([self isCancelled])
   {
      // Must move the operation to the finished state if it is canceled.
      [self willChangeValueForKey:@"isFinished"];
      finished = YES;
      [self didChangeValueForKey:@"isFinished"];
      return;
   }
 
   // If the operation is not canceled, begin executing the task.
   [self willChangeValueForKey:@"isExecuting"];
   
   // 在另一个线程上执行main 方法
   [NSThread detachNewThreadSelector:@selector(main) toTarget:self withObject:nil];
   executing = YES;
   [self didChangeValueForKey:@"isExecuting"];
}

- (void)main {
   @try {
 
       // Do the main work of the operation here.
 
       [self completeOperation];
   }
   @catch(...) {
      // Do not rethrow exceptions.
   }
}
 
- (void)completeOperation {
    [self willChangeValueForKey:@"isFinished"];
    [self willChangeValueForKey:@"isExecuting"];
 
    executing = NO;
    finished = YES;
 
    [self didChangeValueForKey:@"isExecuting"];
    [self didChangeValueForKey:@"isFinished"];
}
 
 
- (BOOL)isConcurrent {
    return YES;
}


- (BOOL)isExecuting {
    return executing;
}
 
- (BOOL)isFinished {
    return finished;
}

@end

```
注意：即使operation 对象是被取消的，也需要通知这个对象的观察者它已经完成了。因为如果一个operation 对象依赖于其他对象，那么它会观察其他对象的`isFinished`属性，只有当它所依赖的对象都已经发出了完成的信号后，这个对象才会开始运行。所以如果忘记发出完成通知，那么可能就有一个operation 对象永远都无法执行。

### 自定义operation 对象的执行行为

#### 配置对象间的依赖关系
前面多次提到，一个operation 对象可能依赖于另外几个operation 对象。在它所依赖的一个operation 对象都运行完成后，这个operation 对象才可以开始运行。

为一个operation 对象配置依赖的操作很简单，只需要调用下面这个函数：

    - (void) addDependency:(NSOperation *)op;

#### 改变operation 对象的执行优先级
对于队列中的operation 对象，它们的执行顺序取决于它们各自的优先级。更准确的说，对于那些已经ready 的operation 对象来说，优先级高的先执行，优先级低的后执行。

operation 对象的优先级用这个函数来设置：

    - (void) setQueuePriority:(NSOperationQueuePriority) priority;

priority 的可能取值有这几个：

* NSOperationQueuePriorityVeryLow
* NSOperationQueuePriorityLow
* NSOperationQueuePriorityNormal
* NSOperationQueuePriorityHigh
* NSOperationQueuePriorityVeryHigh

#### 设置完成时的block
当operation 对象执行完它的任务后，可以用`setCompletionBlock:` 这个函数来设置接下来的工作。

    - (void) setCompletionBlock:(void ^(void))block

### 执行operations
当完成了对operation 对象的配置后，就可以执行了。
#### 使用OperationQueue
执行Operation 的最简单的方法是使用`NSOperationQueue` 。它也是我们分析`SDWebImage` 时所关注的一个核心类。


```
NSOperationQueue* aQueue = [[NSOperationQueue alloc] init];

[aQueue addOperation:anOp]; // Add a single operation
[aQueue addOperations:anArrayOfOps waitUntilFinished:NO]; // Add multiple operations
[aQueue addOperationWithBlock:^{
   /* Do something. */
}];
```

> Never modify an operation object after it has been added to a queue. While waiting in a queue, the operation could start executing at any time, so changing its dependencies or the data it contains could have adverse effects. If you want to know the status of an operation, you can use the methods of the NSOperation class to determine if the operation is running, waiting to run, or already finished.

如果使用`setMaxConcurrentOperationCount:` 将一个operationQueue 的最大可并行operation 数设置为1， 那么这个队列一次只能执行一个operation。

#### 取消operations 
`cancelAllOperations` 可以用来取消一个operationQueue 内的所有operation。
    
    [operationQueue cancelAllOperations];

#### 等待operations 完成
在一个operation 实例上调用`waitUntilFinished`可以短暂地阻塞这个operation，直到此opeartion 执行完。但是通常应该避免这样做。尤其是不要在主线程上调用这个方法，否则可能阻塞主线程，导致程序失去响应。

#### 阻塞operation
在operationQueue 上调用`setSuspended` 可以阻塞那些还没有执行的operation，但是不会影响那些正在执行的和已经执行的operation。

## SDWebImage 中的实现
`SDWebImage` 中，使用NSOperation 对象和GCD 的地方主要有两处，一处是在`SDWebImageDownloader` 中使用`SDWebImageDownloaderOperation` 来完成图片的**下载**任务；一处是在`SDWebImageManager` 使用`SDWebImageCombinedOperation`来完成下载后图片的**缓存**任务。我们只分析`SDWebImageDownloader` 中的多线程策略。

### SDWebImageDownloader
在这个类的类扩展中，有几个属性是与Operation 和OperationQueue 对象有联系的：

```
@property (strong, nonatomic, nonnull) NSOperationQueue *downloadQueue;
@property (weak, nonatomic, nullable) NSOperation *lastAddedOperation;

// This queue is used to serialize the handling of the network responses of all the download operation in a single queue
@property (SDDispatchQueueSetterSementics, nonatomic, nullable) dispatch_queue_t barrierQueue;
```

从名字和注释中不难猜测到它们各自的作用：

* downloadQueue：用来管理 download 操作的队列。
* lastAddedOperation：上一个被添加进队列的 operation。
* barrierQueue：这是一个dispatch_queue_t 类型的属性，作用是串行地处理所有 download 操作的网络响应。

#### SDWebImageDownloaderOperation
`SDWebImageDownloaderOperation` 是`NSOperation` 的子类，是`SDWebImage` 中完成下载任务的operation 对象。这个operation 对象的设计，符合我们在『实现自定义的NSOperation 子类』一节中介绍的各种要求。对于它的start 和其他方法，由于是对网络请求部分的具体实现，与本文主题无关，因此暂时不做介绍（日后若有时间，我将专门写一篇博客来介绍`SDWebImage` 中的网络部分）。

#### downloadQueue
在`downloadImageWithURL:options:progress:completed:`这个方法中，`SDWebImageDownloader` 为每一个下载图片的请求创建一个`SDWebImageDownloaderOperation` ，并将这个`SDWebImageDownloaderOperation` 加入downloadQueue 中。

```
SDWebImageDownloaderOperation *operation = [[sself.operationClass alloc] initWithRequest: request inSession: sself.session options: options];


// 根据传入的SDWebImageDownloaderOptions 来设置当前operation 的优先级
if (options & SDWebImageDownloaderHighPriority) {
    operation.queuePriority = NSOperationQueuePriorityHigh;
} else if (options & SDWebImageDownloaderLowPriority) {
    operation.queuePriority = NSOperationQueuePriorityLow;
}

[sself.downloadQueue addOperation:operation];
```

在将operation 添加进downloadQueue 后，还可以设置operation 对象在这个队列中的执行顺序：

```
if (sself.executionOrder == SDWebImageDownloaderLIFOExecutionOrder) {
    [sself.lastAddedOperation addDependency:operation];
    sself.lastAddedOperation = operation;
}
```
downloadQueue 的默认执行顺序是

* `SDWebImageDownloaderFIFOExecutionOrder`，也就是先入先出，其行为类似于一个队列；

当执行顺序改为

*  `SDWebImageDownloaderLIFOExecutionOrder` 时，则是后入先出，其行为类似于一个栈。`lastAddedOperation` 就是用来记录当前栈顶的那个operation 的，当有新的operation 要进栈时，就把这个新的operation 设置为`lastAddedOperation` 的依赖，这样的话，只有新的operation 执行完毕，`lastAddedOperation` 才能开始执行，因此就实现了后入先出的效果。

#### barrierQueue
前面提到，`barrierQueue` 是用来串行地处理所有的download 操作的网络响应的。关于`dispatch_queue_t` 和`NSOperation` 的对比，可以参见[这儿](http://stackoverflow.com/questions/10373331/nsoperation-vs-grand-central-dispatch)。
我们来看下具体的操作过程是什么样子的。


```
- (nullable SDWebImageDownloadToken *)addProgressCallback:(SDWebImageDownloaderProgressBlock)progressBlock
                                           completedBlock:(SDWebImageDownloaderCompletedBlock)completedBlock
                                                   forURL:(nullable NSURL *)url
                                           createCallback:(SDWebImageDownloaderOperation *(^)())createCallback {
    // The URL will be used as the key to the callbacks dictionary so it cannot be nil. If it is nil immediately call the completed block with no image or data.
    if (url == nil) {
        if (completedBlock != nil) {
            completedBlock(nil, nil, nil, NO);
        }
        return nil;
    }

    __block SDWebImageDownloadToken *token = nil;

    dispatch_barrier_sync(self.barrierQueue, ^{
        SDWebImageDownloaderOperation *operation = self.URLOperations[url];
        if (!operation) {
            operation = createCallback();
            self.URLOperations[url] = operation;

            __weak SDWebImageDownloaderOperation *woperation = operation;
            operation.completionBlock = ^{
              SDWebImageDownloaderOperation *soperation = woperation;
              if (!soperation) return;
              if (self.URLOperations[url] == soperation) {
                  [self.URLOperations removeObjectForKey:url];
              };
            };
        }
        id downloadOperationCancelToken = [operation addHandlersForProgress:progressBlock completed:completedBlock];

        token = [SDWebImageDownloadToken new];
        token.url = url;
        token.downloadOperationCancelToken = downloadOperationCancelToken;
    });

    return token;
}


- (void)cancel:(nullable SDWebImageDownloadToken *)token {
    dispatch_barrier_async(self.barrierQueue, ^{
        SDWebImageDownloaderOperation *operation = self.URLOperations[token.url];
        BOOL canceled = [operation cancel:token.downloadOperationCancelToken];
        if (canceled) {
            [self.URLOperations removeObjectForKey:token.url];
        }
    });
}
```
这段代码中，`SDWebImageDownloader` 维护了一个NSDictionary 用以将url 和url 对应的operation 联系起来。值得注意的是，在前一个方法中，使用了`dispatch_barrier_async`，在后一个方法中，则使用了`dispatch_barrier_sync`，那么这两个函数究竟有什么区别呢？

我们知道`dispatch_barrier_async` 和`dispatch_barrier_sync`都是为了解决竞争问题而提出来的。举例来说，现在我们在项目中使用一个数据库，那么对数据库的操作就有读和写两种操作。假设我们需要先读两次数据库，然后再写入一个数据，然后再读出两个数据，那么显然读和写的操作是不能同时发生的，如果同时发生，就有可能造成数据错误。

使用`dispatch_barrier_(a)sync`，上述任务可以这样实现：

    dispatch_async(queue, blk1_read);
    dispatch_async(queue, blk2_read);
    dispatch_async(queue, blk3_read);
    dispatch_barrier_async(queue, blk_write);
    dispatch_async(queue, blk4_read);
    dispatch_async(queue, blk5_read);
    
`dispatch_barrier_(a)sync` 就像是一个屏障一样，它确保blk_write 一定会等到blk1，blk2，blk3 全部执行完毕才开始执行，在blk_write 执行完毕后，blk4 和blk5 又正常地并行运行。

`sync` 和`async` 的区别体现在：将block 追加到指定的queue 中是否是同步的。如果是`async` 的，那么`dispatch_barrier_async` 不会等待追加操作结束，当前进程会继续进行；如果是`sync` 的，那么`dispatch_barrier_async` 会等待一直追加操作结束，当前进程会因此阻塞。

用例子来解释：

    // A
    // step1
    dispatch_barrier_async(queue, ^{
        step2
    });
    // step3
    
    // B
    // step4
    dispatch_barrier_sync(queue, ^{
        // step5
    });
    // step6
    
在A 中，step1 先执行，但是step3 可能会在step2 之前执行完，因此执行顺序可能是`step1->step2->step3`，也可能是`step1->step3->step2`；在B 中，执行的顺序则一定是`step4->step5->step6`。

在`addProgressCallback:`方法中，因为需要返回一个token，而这个token 是在block 中计算的，所以我们一定要等到block 的追加操作完成才可以返回，否则就会得到nil，因此需要使用`dispatch_barrier_sync`；而在`cancel:` 方法中，不需要等待，因此使用`dispatch_barrier_async` 就可以了。


## 总结
多线程策略无疑是iOS 应用开发中一个十分重要的考虑因素。随着项目的进行，需求不断增多，应用的逻辑越加复杂，对性能的要求也随之提要，因此我们不得不借助于多线程来适应这些要求。本文介绍了`SDWebImage` 这个库中多线程策略的选择和实现，在分析过程中，我发觉自己之前写的代码都太小儿科了，也感受到：能够正确地进行多线程编程，应该是一个iOS 开发者必要的能力。

另外，为了写出这篇文章，我又重新研读了几遍`SDWebImage` 的代码，感受和在写前一篇文章时果然有很多不同。看来还是要多读优秀的源代码，而且要多加思考和总结。








 










