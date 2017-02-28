#NSFileManager

###判断文件是否存在

    NSFileManager *fileManager = [NSFileManager defaultManager];
    
    NSString *documentsPath = [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask,YES) firstObject];
    
    NSString *filePath = [documentsPath stringByAppendingPathComponent:@"file.txt"];
    
    BOOL fileExists = [fileManager fileExistsAtPath:filePath];
    
###列出目录下的所有文件

    NSFileManager *fileManager = [NSFileManager defaultManager];
    
    NSURL *bundleURL = [[NSBundle mainBundle] bundleURL];
    
    NSArray *contents = [fileManager contentsOfDirectoryAtURL:bundleURL
    includingPropertiesForKeys:@[]
    options:NSDirectoryEnumerationSkipsHiddenFiles
    error:nil];
    
    NSPredicate *predicate = [NSPredicate predicateWithFormat:
        @"pathExtension ENDSWITH '.png'"];
    
    for (NSString *path in [contents filteredArrayUsingPredicate:predicate])
    {
        // Enumerate each .png file in directory
    }
    

###递归地列出目录下的所有文件
    
    NSFileManager *fileManager = [NSFileManager defaultManager];
    
    NSURL *bundleURL = [[NSBundle mainBundle] bundleURL];
    
    NSDirectoryEnumerator *enumerator = [fileManager enumeratorAtURL:bundleURL
        includingPropertiesForKeys:@[NSURLNameKey, NSURLIsDirectoryKey]
        options:NSDirectoryEnumerationSkipsHiddenFiles
        errorHandler:^BOOL(NSURL *url, NSError *error)
    {
        NSLog(@"[Error] %@ (%@)",error,url);
    }];
    
    NSMutableArray *mutableFileURLs = [NSMutableArray array];
    
    for (NSURL *fileURL in enumerator)
    {
        NSString *filename;
        [fileURL getResourceValue:&filename
            forKey:NSURLNameKey
            error:nil];
        
        NSNumber *isDirectory;
        [fileURL getResourceValue:&isDirectory
            forKey:NSURLIsDirectoryKey
            error:nil];
        
        // Skip directories with '_' prefix
        if([isDirectory boolValue] && [fileName hasPrefix:@"_"])
        {
            [enumerator skipDescendants];
            continue;
        }
        
        if(! [isDirectory boolValue])
        {
            [mutableFileURLs addObject:fileURL];
        }
    }
    

###创建一个目录

    NSFileManager *fileManager = [NSFileManager defaultManager];
    
    NSString *documentsPath = [NSSearchPathForDirectoriesInDomains(
        NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
    
    NSString *imagesPath = [documentsPath stringByAppendingPathComponent:@"images"];
    
    if(![fileManager fileExistsAtPath:imagesPath])
    {
        [fileManager createDirectoryAtPath:imagesPath
                withIntermediateDirectories:NO
                attributes:nil
                error:nil];
    }
    
###删除一个文件

    NSFileManager *fileManager = [NSFileManager defaultManager];
    
    NSString *documentsPath = [NSSearchPathForDirectoriesInDomains(
        NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
        
    NSString *filePath = [documentsPath stringByAppendingPathComponent:@"image.png"];
    
    NSError *error = nil;
    if (! [fileManager removeItemAtPath:filePath
                    error:&error])
    {
        NSLog(@"[Error] %@ (%@)",error,filePath);
    }
    
###查看一个文件的创建日期

    NSFileManager *fileManager = [NSFileManager defaultManager];
    
    NSString *documentsPath = [NSSearchPathForDirectoriesInDomains(
        NSDocumentDirectory, NSUserDomainMask, YES) firstObject];
        
    NSString *filePath = [documentsPath stringByAppendingPathComponent:@"Document.pages"];
    
    NSDate *creationDate = nil;
    
    if([fileManager fileExistsAtPath:filePath]) 
    {
        NSDictionary *attributes = [fileManager attributesOfItemAtPath:filePath
            error:nil];
        creationDate = attributes[NSFileCreationDate];
    }
    
下面是可以通过上述方法获得的文件属性表：

| 键 | 属性 |
|:---|:---|
|NSFileAppendOnly|文件是否是只读的|
|NSFileBusy|文件是否被其他程序占用（忙）|
|NSFileCreationDate|文件的创建日期|
|NSFileOwnerAccountName|文件拥有者的名称|
|NSFileGroupOwnerAccountName|文件拥有者所在组的名称|
|NSFileDeviceIdentifier|文件所在的设备的标识符|
|NSFileExtensionHidden|文件的扩展符是否是隐藏的|
|NSFileImmutable|文件是否是可变的|
|NSFileModificationDate|文件的最新修改日期|
|NSFileSize|文件的大小|
|NSFileType|文件的类型|
    
    
    
    
    