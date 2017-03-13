# SDWebImage 源代码剖析-缓存策略
在上一篇文章中，我们对`Masonry` 的源代码进行了分析。今天我们将对另外一个在iOS 开发中广泛使用的库的源代码进行分析，这个库就是鼎鼎大名的`SDWebImage`。事实上`SDWebImage` 是如此常用和好用，以至于没有听说过没有用过没有分析过这个库的人应该是少数了。下面是我个人对这个库的缓存策略部分的代码分析和理解。由于本人才疏学浅，若有错误，请大家及时指出，不吝赐教。

## 使用方法
`SDWebImage` 的使用非常简洁，往往可以用一行代码来完成图片设置工作。下面列出一些常用设置方法。

    - (void) sd_setImageWithURL:(nullable NSURL *) url;
    - (void) sd_setImageWithURL:(nullable NSURL *) url placeholderImage:(nullable UIImage *) placeholder
    - (void) sd_setImageWithURL:(nullable NSURL *) url placeholderImage:(nullable UIImage *) placeholder options:(SDWebImageOptions) options;

    
    - (void）sd_setImageWithURL:(nullable NSURL *) url completed:(nullable SDExternalCompletionBlock) completedBlock;
    - (void) sd_setImageWithURL:(nullable NSURL *) url placeholderImage:(nullable UIImage *) placeholder options:(SDWebImageOptions) options completed:(nullable SDExternalCompletionBlock) completedBlock;

    
    
url 是远程图片的url 地址，placeholderImage 是远程图片尚未下载完成时显示的占位图片，completedBlock 是远程图片下载完成后将要执行的block，options 是一组NS_OPTIONS枚举值：
    
```
typedef NS_OPTIONS(NSUInteger, SDWebImageOptions) {

    // 当按照给出的url 下载失败后，这个url 会被加入黑名单，
    // 如果下次这个url 再次出现，就不会尝试去下载
    SDWebImageRetryFailed = 1 << 0,
    
    // 通常来说，图像下载是在UI交互过程中进行的，
    // 如果使用这个flag 的话就会延迟图片的下载
    SDWebImageLowPriority = 1 << 1,
    
    // 禁止磁盘缓存，只允许内存缓存 
    SDWebImageCacheMemoryOnly = 1 << 2,
    
    // 允许渐进式加载。默认的是加载完成才显示
    SDWebImageProgressiveDownload = 1 << 3,
    
    // 磁盘缓存将会由NSURLCache 而不是SDWebImage 来处理，因此可能带来轻微的性能下降。
    // 使用于使用固定的图片url 但是图片内容可能变化的场景
    SDWebImageRefreshCached = 1 << 4,
    
    // 如果应用进入后台状态，继续图片下载，应用因此将会额外活跃一段时间，
    // 如果这段时间用完但是下载任务尚未完成，那么下载就会被取消
    SDWebImageContinueInBackground = 1 << 5,
    
    // 处理存储在NSHTTPCookieStore 中的cookie
    SDWebImageHandleCookie = 1 << 6,
    
    // 允许不受信任的SSL认证。通常用于测试环境，很少用于生产环境
    SDWebImageAllowInvalidSSLCertificates = 1 << 7,
    
    // 提高该图片下载的优先级
    SDWebImageHighPriority = 1 << 8,
    
    // 通常在图片加载时都会显示placeholder。但是这个flag 会将placeholder 的显示延迟到
    // 图片加载之后（不是很懂这个选项的意思）
    /**
     * By default, placeholder images are loaded while the image is loading. This flag will delay the loading
     * of the placeholder image until after the image has finished loading.
     */
    SDWebImageDelayPlaceholder = 1 << 9,
    
    // 并不常用的方法。用于对下载的图片进行变换。
    // 这个变换工作由实现了transformDownloadedImage 的协议的类完成
    SDWebImageTransformAnimatedImage = 1 << 10,
    
    // 在下载完成之后将图片设置成imageView.image 之前，
    // 允许你对下载的图片进行额外的处理
    SDWebImageAvoidAutoSetImage = 1 << 11,
    
    // 图片默认会被解码成它们的原始尺寸。这个flag 会将图片按照设备的内存来进行缩放。
    // 如果SDWebImageProgressiveDownload 被设置了，那么这个选项就不起作用 
    SDWebImageScaleDownLargeImages = 1 << 12
};
    
```

## 内部实现
### UIView+WebCache
上面列出的方法其实是一个核心方法接受不同参数时的不同版本。

    - (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                 completed:(nullable SDExternalCompletionBlock)completedBlock;
                 
这个核心方法内部是这样实现的：
    
    - (void)sd_setImageWithURL:(nullable NSURL *)url
          placeholderImage:(nullable UIImage *)placeholder
                   options:(SDWebImageOptions)options
                  progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                 completed:(nullable SDExternalCompletionBlock)completedBlock {
    [self sd_internalSetImageWithURL:url
                    placeholderImage:placeholder
                             options:options
                        operationKey:nil
                       setImageBlock:nil
                            progress:progressBlock
                           completed:completedBlock];
    }


进入`sd_internalSetImageWithURL:` 这个方法的内部，我们总算看到了一段期望中的复杂代码：

```
// UIView+WebCache.m
- (void)sd_internalSetImageWithURL:(nullable NSURL *)url
                  placeholderImage:(nullable UIImage *)placeholder
                           options:(SDWebImageOptions)options
                      operationKey:(nullable NSString *)operationKey
                     setImageBlock:(nullable SDSetImageBlock)setImageBlock
                          progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                         completed:(nullable SDExternalCompletionBlock)completedBlock {
    // 1
    NSString *validOperationKey = operationKey ?: NSStringFromClass([self class]); 
    
    // 2
    [self sd_cancelImageLoadOperationWithKey:validOperationKey];
    
    // 3
    objc_setAssociatedObject(self, &imageURLKey, url, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
    
    // 4
    if (!(options & SDWebImageDelayPlaceholder)) {
        dispatch_main_async_safe(^{
            [self sd_setImage:placeholder imageData:nil basedOnClassOrViaCustomSetImageBlock:setImageBlock];
        });
    }
    
    if (url) {
        // check if activityView is enabled or not
        if ([self sd_showActivityIndicatorView]) {
            [self sd_addActivityIndicator];
        }
        
        __weak __typeof(self)wself = self;
        
        // 5
        id <SDWebImageOperation> operation = [SDWebImageManager.sharedManager loadImageWithURL:url options:options progress:progressBlock completed:^(UIImage *image, NSData *data, NSError *error, SDImageCacheType cacheType, BOOL finished, NSURL *imageURL) {
            __strong __typeof (wself) sself = wself;
            [sself sd_removeActivityIndicator];
            if (!sself) {
                return;
            }
            dispatch_main_async_safe(^{
                if (!sself) {
                    return;
                }
                // 6
                if (image && (options & SDWebImageAvoidAutoSetImage) && completedBlock) {
                    completedBlock(image, error, cacheType, url);
                    return;
                } else if (image) {
                
                    // 7
                    [sself sd_setImage:image imageData:data basedOnClassOrViaCustomSetImageBlock:setImageBlock];
                    [sself sd_setNeedsLayout];
                } else {
                    // 8
                    if ((options & SDWebImageDelayPlaceholder)) {
                        [sself sd_setImage:placeholder imageData:nil basedOnClassOrViaCustomSetImageBlock:setImageBlock];
                        [sself sd_setNeedsLayout];
                    }
                }
                if (completedBlock && finished) {
                    completedBlock(image, error, cacheType, url);
                }
            });
        }];
        
        // 9
        [self sd_setImageLoadOperation:operation forKey:validOperationKey];
    } else {
        dispatch_main_async_safe(^{
            [self sd_removeActivityIndicator];
            if (completedBlock) {
                NSError *error = [NSError errorWithDomain:SDWebImageErrorDomain code:-1 userInfo:@{NSLocalizedDescriptionKey : @"Trying to load a nil url"}];
                completedBlock(nil, error, SDImageCacheTypeNone, url);
            }
        });
    }
}
```                

1. 用当前视图的类名来作为一个key。`SDWebImage` 不仅能用来设置UIImageView，也可以用来设置UIButton。
2. 在当前视图的operationDictionary 中进行查找，如果已经有key为operationKey 的operation，则取消这个operation。
3. 将该远程图片的url 与当前视图的imageURLKey 用关联对象设置在一起。关于关联对象，网上也已经有很多不错的分析文章。
4. 如果没有设置`SDWebImageDelayPlaceholder`这个选项，那么就先将当前视图设置成placeholder。
5. 使用`SDWebImageManager`的`loadImageWithURL:` 创建一个operation 对象。
6. 如果image下载完成了，并且设置了`SDWebImageAvoidAutoSetImage`选项，而且传入了对下载的图片进行处理的block，那么就进行对应处理。
7. 如果image下载完成了，没有额外处理要求，那么将当前视图设置为image。
8. 如果image下载失败了，那么还是将当前视图设置为placeholder。
9. 将5中创建的operation 的key 设置为operationKey，然后加入当前视图的operationDictionary 中。

### SDWebImageManager
由代码可知，operation 这个对象是设置过程的核心与关键。既然它是`SDWebImageManager`创建的，我们自然要去探究下`SDWebImageManager`的内部秘密。


```
- (id <SDWebImageOperation>)loadImageWithURL:(nullable NSURL *)url
                                     options:(SDWebImageOptions)options
                                    progress:(nullable SDWebImageDownloaderProgressBlock)progressBlock
                                   completed:(nullable SDInternalCompletionBlock)completedBlock {
                                   
    // 1
    // Invoking this method without a completedBlock is pointless
    NSAssert(completedBlock != nil, @"If you mean to prefetch the image, use -[SDWebImagePrefetcher prefetchURLs] instead");

    ... 
    // 2
    @synchronized (self.runningOperations) {
        [self.runningOperations addObject:operation];
    }
    
    // 3
    NSString *key = [self cacheKeyForURL:url];
    
    // 4
    ...
}
```

#### 1
    
```
// Invoking this method without a completedBlock is pointless
    NSAssert(completedBlock != nil, @"If you mean to prefetch the image, use -[SDWebImagePrefetcher prefetchURLs] instead");

    // Very common mistake is to send the URL using NSString object instead of NSURL. For some strange reason, Xcode won't
    // throw any warning for this type mismatch. Here we failsafe this error by allowing URLs to be passed as NSString.
    if ([url isKindOfClass:NSString.class]) {
        url = [NSURL URLWithString:(NSString *)url];
    }

    // Prevents app crashing on argument type error like sending NSNull instead of NSURL
    if (![url isKindOfClass:NSURL.class]) {
        url = nil;
    }

    __block SDWebImageCombinedOperation *operation = [SDWebImageCombinedOperation new];
    __weak SDWebImageCombinedOperation *weakOperation = operation;

    BOOL isFailedUrl = NO;
    if (url) {
        @synchronized (self.failedURLs) {
            isFailedUrl = [self.failedURLs containsObject:url];
        }
    }

    if (url.absoluteString.length == 0 || (!(options & SDWebImageRetryFailed) && isFailedUrl)) {
        [self callCompletionBlockForOperation:operation completion:completedBlock error:[NSError errorWithDomain:NSURLErrorDomain code:NSURLErrorFileDoesNotExist userInfo:nil] url:url];
        return operation;
    }
```
首先进行异常处理。这是编程时一个常见的习惯，将可能遇到的各种问题和对应的解决方案放在方法的开头，可以使得逻辑变得清晰，同时也避免了无谓的函数调用开销。SDWebImage 团队贴心地为我们处理了常见的误将NSString 类型的对象传入NSURL 类型的参数的错误。这启示我们，在编写自己的库时，应尽可能考虑到各种常见错误，并对它们进行处理，这样可以使得你的库对于别的开发者更加友好。

#### 2
将operation 加入`SDWebImageManager` 的runningOperations 数组中。

#### 3
获得远程图片url 所对应的key。

    - (nullable NSString *)cacheKeyForURL:(nullable NSURL *)url {
    if (!url) {
        return @"";
    }
    
    if (self.cacheKeyFilter) {
        // 如果定义了用来对url 进行过滤的filter，那么就用filter 来处理
        return self.cacheKeyFilter(url);
    } else {
        // 否则就返回url 的string表示
        return url.absoluteString;
    }
    }
    
#### 4
先查询operation 的缓存操作。

```
- (nullable NSOperation *)queryCacheOperationForKey:(nullable NSString *)key done:(nullable SDCacheQueryCompletedBlock)doneBlock {
    if (!key) {
        if (doneBlock) {
            doneBlock(nil, nil, SDImageCacheTypeNone);
        }
        return nil;
    }

    // First check the in-memory cache...   
    // 首先检查该图片在内存中是否有缓存
    UIImage *image = [self imageFromMemoryCacheForKey:key];
    if (image) {
        NSData *diskData = nil;
        if ([image isGIF]) {
            diskData = [self diskImageDataBySearchingAllPathsForKey:key];
        }
        if (doneBlock) {
            doneBlock(image, diskData, SDImageCacheTypeMemory);
        }
        // 如果在内存中有缓存，这里就会直接返回了
        return nil;
    }

    // 当该图片在内存中没有缓存的时候才会执行下面的代码
    NSOperation *operation = [NSOperation new];
    dispatch_async(self.ioQueue, ^{
        if (operation.isCancelled) {
            // do not call the completion if cancelled
            return;
        }

        @autoreleasepool {
        
            // 获得该图片在磁盘中的缓存
            NSData *diskData = [self diskImageDataBySearchingAllPathsForKey:key];
            UIImage *diskImage = [self diskImageForKey:key];
            if (diskImage && self.config.shouldCacheImagesInMemory) {
                NSUInteger cost = SDCacheCostForImage(diskImage);
                // 将磁盘缓存保存在内存中
                [self.memCache setObject:diskImage forKey:key cost:cost];
            }

            if (doneBlock) {
                dispatch_async(dispatch_get_main_queue(), ^{
                    doneBlock(diskImage, diskData, SDImageCacheTypeDisk);
                });
            }
        }
    });

    return operation;
}
```

在查询到operation 的缓存操作后，设置doneBlock:

```
operation.cacheOperation = [self.imageCache queryCacheOperationForKey:key done:^(UIImage *cachedImage, NSData *cachedData, SDImageCacheType cacheType) {
        if (operation.isCancelled) {
            [self safelyRemoveOperationFromRunning:operation];
            return;
        }
        // 1
        if ((!cachedImage || options & SDWebImageRefreshCached) && (![self.delegate respondsToSelector:@selector(imageManager:shouldDownloadImageForURL:)] || [self.delegate imageManager:self shouldDownloadImageForURL:url])) {
        
            // 2
            if (cachedImage && options & SDWebImageRefreshCached) {
                // If image was found in the cache but SDWebImageRefreshCached is provided, notify about the cached image
                // AND try to re-download it in order to let a chance to NSURLCache to refresh it from server.
                [self callCompletionBlockForOperation:weakOperation completion:completedBlock image:cachedImage data:cachedData error:nil cacheType:cacheType finished:YES url:url];
            }

            // download if no image or requested to refresh anyway, and download allowed by delegate
            // 3
            SDWebImageDownloaderOptions downloaderOptions = 0;
            if (options & SDWebImageLowPriority) downloaderOptions |= SDWebImageDownloaderLowPriority;
            if (options & SDWebImageProgressiveDownload) downloaderOptions |= SDWebImageDownloaderProgressiveDownload;
            if (options & SDWebImageRefreshCached) downloaderOptions |= SDWebImageDownloaderUseNSURLCache;
            if (options & SDWebImageContinueInBackground) downloaderOptions |= SDWebImageDownloaderContinueInBackground;
            if (options & SDWebImageHandleCookies) downloaderOptions |= SDWebImageDownloaderHandleCookies;
            if (options & SDWebImageAllowInvalidSSLCertificates) downloaderOptions |= SDWebImageDownloaderAllowInvalidSSLCertificates;
            if (options & SDWebImageHighPriority) downloaderOptions |= SDWebImageDownloaderHighPriority;
            if (options & SDWebImageScaleDownLargeImages) downloaderOptions |= SDWebImageDownloaderScaleDownLargeImages;
            
            // 4
            if (cachedImage && options & SDWebImageRefreshCached) {
                // force progressive off if image already cached but forced refreshing
                downloaderOptions &= ~SDWebImageDownloaderProgressiveDownload;
                // ignore image read from NSURLCache if image if cached but force refreshing
                downloaderOptions |= SDWebImageDownloaderIgnoreCachedResponse;
            }
            
            // 5
            SDWebImageDownloadToken *subOperationToken = [self.imageDownloader downloadImageWithURL:url options:downloaderOptions progress:progressBlock completed:^(UIImage *downloadedImage, NSData *downloadedData, NSError *error, BOOL finished) {
                __strong __typeof(weakOperation) strongOperation = weakOperation;
                if (!strongOperation || strongOperation.isCancelled) {
                    // Do nothing if the operation was cancelled
                    // See #699 for more details
                    // if we would call the completedBlock, there could be a race condition between this block and another completedBlock for the same object, so if this one is called second, we will overwrite the new data
                } else if (error) {
                    [self callCompletionBlockForOperation:strongOperation completion:completedBlock error:error url:url];

                    // 6
                    if (   error.code != NSURLErrorNotConnectedToInternet
                        && error.code != NSURLErrorCancelled
                        && error.code != NSURLErrorTimedOut
                        && error.code != NSURLErrorInternationalRoamingOff
                        && error.code != NSURLErrorDataNotAllowed
                        && error.code != NSURLErrorCannotFindHost
                        && error.code != NSURLErrorCannotConnectToHost
                        && error.code != NSURLErrorNetworkConnectionLost) {
                        @synchronized (self.failedURLs) {
                            [self.failedURLs addObject:url];
                        }
                    }
                }
                else {
                    if ((options & SDWebImageRetryFailed)) {
                        @synchronized (self.failedURLs) {
                            [self.failedURLs removeObject:url];
                        }
                    }
                    
                    
                    BOOL cacheOnDisk = !(options & SDWebImageCacheMemoryOnly);

                    // 7
                    if (options & SDWebImageRefreshCached && cachedImage && !downloadedImage) {
                        // Image refresh hit the NSURLCache cache, do not call the completion block
                    } else if (downloadedImage && (!downloadedImage.images || (options & SDWebImageTransformAnimatedImage)) && [self.delegate respondsToSelector:@selector(imageManager:transformDownloadedImage:withURL:)]) {
                        
// 8
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
                            UIImage *transformedImage = [self.delegate imageManager:self transformDownloadedImage:downloadedImage withURL:url];

                            if (transformedImage && finished) {
                                BOOL imageWasTransformed = ![transformedImage isEqual:downloadedImage];
                                // pass nil if the image was transformed, so we can recalculate the data from the image
                                [self.imageCache storeImage:transformedImage imageData:(imageWasTransformed ? nil : downloadedData) forKey:key toDisk:cacheOnDisk completion:nil];
                            }
                            
                            [self callCompletionBlockForOperation:strongOperation completion:completedBlock image:transformedImage data:downloadedData error:nil cacheType:SDImageCacheTypeNone finished:finished url:url];
                        });
                    } else {
                        if (downloadedImage && finished) {
                            [self.imageCache storeImage:downloadedImage imageData:downloadedData forKey:key toDisk:cacheOnDisk completion:nil];
                        }
                        [self callCompletionBlockForOperation:strongOperation completion:completedBlock image:downloadedImage data:downloadedData error:nil cacheType:SDImageCacheTypeNone finished:finished url:url];
                    }
                }

                if (finished) {
                    [self safelyRemoveOperationFromRunning:strongOperation];
                }
            }];
            operation.cancelBlock = ^{
                [self.imageDownloader cancel:subOperationToken];
                __strong __typeof(weakOperation) strongOperation = weakOperation;
                [self safelyRemoveOperationFromRunning:strongOperation];
            };
        } else if (cachedImage) {
        
            // 9
            __strong __typeof(weakOperation) strongOperation = weakOperation;
            [self callCompletionBlockForOperation:strongOperation completion:completedBlock image:cachedImage data:cachedData error:nil cacheType:cacheType finished:YES url:url];
            [self safelyRemoveOperationFromRunning:operation];
        } else {
            // Image not in cache and download disallowed by delegate
            
            // 10
            __strong __typeof(weakOperation) strongOperation = weakOperation;
            [self callCompletionBlockForOperation:strongOperation completion:completedBlock image:nil data:nil error:nil cacheType:SDImageCacheTypeNone finished:YES url:url];
            [self safelyRemoveOperationFromRunning:operation];
        }
    }];

```
1. 由if条件，假设后一条件成立，如果缓存存在，那么只有在设置了`SDWebImageRefreshCache`才会进入if 语句体中；如果缓存不存在，那么肯定会进入if 语句体中。
2. 如果说缓存存在，而且设置了`SDWebImageRefreshCache`，那么就应该从server 上重新下载图片以更新本地缓存。
3. 设置下载时的选项。
4. 如果设置了`SDWebImageRefreshCache`，那么必须取消渐进式下载，而且还要忽略从NSURLCache 中获得的缓存响应。（此处我还没有理解到底是怎么回事，等了解了NSURLCache 的知识后再来填这个坑）
5. 调用imageDownloader 进行图片下载。
6. 如果发生了错误，且错误原因不是列出来的这些原因中的一个，那么就把这个url 加入黑名单中。
7. 如果缓存存在，设置了`SDWebImageRefreshCache`，而且downloadedImage为nil，那么说明命中了
NSURLCache 缓存，什么事也不做。
8. 如果downloadedImage非nil，并且设置了`SDWebImageTransformAnimatedImage`，那么就在主线程中对图片进行变换，然后将变换后的图片存储在内存中或内存和磁盘上。如果不需要变换，那么直接将downloadedImage 存储在内存中或内存和磁盘上。
9. 如果缓存存在，且其他条件都不成立，那么直接取出缓存中的图片，然后在主线程中更新视图。
10. 如果缓存不存在，并且也不允许下载，那么直接调用completedBlock。

#### 缓存策略小结
我们来小结一下`SDWebImage`的缓存策略。在给一个UIimageView 或者UIButton 设置了远端图片的url 后，`SDWebImage`首先以url 为key 在内存中寻找图片缓存，如果在内存中没找到就会去磁盘中寻找，如果找到了，则将磁盘中的缓存拷贝一份到内存中，然后使用缓存来设置视图。如果在
磁盘和内存中都没有找到，那么才会下载远程图片，然后将远程图片缓存在内存中或者是内存和磁盘上。

## 总结
本文对`SDWebImage` 的源代码进行了分析，其中着重介绍了它的缓存策略。因为篇幅有限，所以对于它的其他方面，如GCD 和NSOperationQueue 的使用，网络请求等知识都没有进行深入研究。在日后的系列文章中，我将回过头来针对这些知识进行再次探索，期待有新的收获。

这一段时间接连分析了`Masonry` 和`SDWebImage` 两个著名开源库的源代码。在我的心目中，`SDWebImage` 才是好代码的典范：架构漂亮，逻辑清晰，注释明确。





