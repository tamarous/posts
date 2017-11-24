# WWDC2013 Hidden Gems in Cocoa and Cocoa Touch
这个Seesion介绍了Cocoa和Cocoa Touch框架中一些鲜为人知的特性，总共有29个tips，从中我挑一些比较常用的介绍一下。视频[地址](https://developer.apple.com/videos/play/wwdc2013/228/)。

## Xcode 相关技巧
* Xcode的Open Quickly功能拥有很强的模糊搜索能力，在我们对一个类或文件印象模糊时，这个功能可以为我们提供最接近最可能的结果。

* Xcode的编辑区的左上角有一个功能按钮，点击这个按钮后我们可以快速导航到最近文件、包含的头文件、父类子类等实现文件中。
![Xcode Tool](http://7xnyik.com1.z0.glb.clouddn.com/Xcode-tool.png-pic)

* Xcode中，可以在断点处右键来编辑这个断点，在弹出来的界面中可以设置诸如“运行100次后停下”、“打印该处变量值”等功能。

* 在调试时我们可能会想查看一个对象的信息，也就是po + 变量名，但这时得到的输出往往不是你想要的，那么我们就可以重写该对象所属类的debugDescription方法来输出一串自定义字符串。注意这个方法是一个实例方法而不是类方法。

* 在调试UI时我们可能会想查看一个UI控件的层次关系，那么除了在Interface Builder里查看外，我们还可以在控制台中查看这个view的所有层次，方法是在控制台中输入：po [self.view recursiveDescription]

## Objective-C 语言相关

* 在使用NSArray或NSDictionary以及对应的可变版本时，可以使用[]来快速访问对应下标的元素，而自定义的类型，只要实现了两个相应方法，其实也可以用下标来访问。
例：

```
// Declaration 
@interface RecordSet: NSObject
@property (strong) NSMutableArray *indexedValues;
- (id) objectAtIndexedSubscript:(NSUInteger) idx;
- (void) setObject:(id) obj atIndexedSubscript:(NSUInteger) idx;
@end

// Implementation
- (id) objectAtIndexedSubscript:(NSUInteger) idx {
    return [self.indexedValues objectAtIndex:idx];
}

- (void) setObject:(id) obj atIndexedSubscript:(NSUInteger) idx {   
    [self.indexedValues insertObject: obj atIndex: idx];
}


```

* 与上面的类似，自定义的类型，只要实现两个相应方法，也可以用键值对的方式来访问。

```
@interface Person: NSObject
@property (strong) NSMutableDictionary *keyedValues;
- (id) objectAtKeyedSubscript: (id<NSCopying>) key;
- (void) setObject:(id) obj forKeyedSubscript: (id<NSCopying>) key:
@end

- (id) objectAtKeyedSubscript: (id<NSCopying>) key {
    return [self.keyedValues objectForKey: key];
}

- (void) setObject:(id) obj forKeyedSubscript: (id<NSCopying>) key {
    [self.keyedValues setObject:obj forKey: key];
}
```

## Foundation 框架
* NSExpression是一个很有用的类，可以将一个字符串当做一个数学表达式来进行处理，用来进行计算器等应用的开发应该会很方便。

例子：

```
NSString *text = @"3 + 5 * 4e10";
NSExpression *e = [NSExpression expressionWithFormat: text, nil];
NSNumber *result = [e expressionValueWithObject: nil context:nil];
NSLog(@"result: %@”, result);
```

* 针对集合类型的一些小技巧：

(1) 快速逆序一个数组。

```
NSArray *numbers = @[@1, @2, @3];
NSArray *reversed = numbers.reverseObjectEnumerator.allObjects;
```

(2) 从一个集合中产生一个可变副本

```
NSArray *unknown = self.values;
NSMutableArray *newArray = [NSMutableArray arrayWithArray: unknown];
```

(3) 声明并遍历一个由不同类型元素组成的集合

```
id<NSFaseEnumeration> collection = values;
for( id object in collection) {

}

```

(4) 遍历一个自定义类型，只要让这个类型实现

```
 - (NSUInteger) countByEnumeratingWithState: (NSFastEnumerationState *)state
 objects:(__unsafe__unretained id []) buffer count:(NSUInteger) len {
    return [self.array countByEnumeratingWithState: state object: buffer count: len];
}
```

* 可以用NSExpression来计算用字符串表示的数学表达式的值：

```
NSString *text = @"3 + 5 * 4";
NSExpression *e = [NSExpression expressionWithFormat: text, nil];
NSNumber *result = [e expressionValueWithObject: nil, context: nil];
NSLog(@"result: %@", result);
```

* NSSet和NSOrderedSet分别可以用来无序集合和有序集合，并且可以对两个集合进行交并补运算。


* 通过@encode 可以将一个任意类型的量或对象包装成一个NSValue对象

```
NSMutableArray *array = [@[] mutableCopy];
array[0] = [NSValue valueWithPoint: CGPointZero];
array[1] = [NSValue valueWithRange: NSMakeRange(3,17)];

typedef struct RGB {
   float red, green, blue;
} _RGB;

RGB color = {1.0f, 0.0f, 0.0f};
array[2] = [NSValue valueWithBytes:&color objCType:@encode(RGB)];
```

* KVC中内置了一些对Collection进行处理的键，使用这些键我们可以快速对Collection整体进行处理而不用先得到Collection中的每个元素并进行处理，然后再把处理结果转为一个Collection了。

```
NSArray *words = @[@"Alpha", @"Bravo", @"Charlie"];

[words valueForKey:@"uppercaseString"];
// @[@"ALPHA", @"BRAVO", @"CHARLIE"];

[words valueForKey:@"length"];
// @[@5, @5, @7];
```
另外还有一些具有特殊功能的键，按照功能可以分为三类：
1. 简单操作：

|符号|返回类型|
|:---:|:---:|
|@count|NSNumber|
|@sum|NSNumber|
|@avg|NSNumber|
|@max|id|
|@min|id|

2. 对象操作：

|符号|返回类型|
|:---:|:---:|
|@unionOfObjects|NSArray|
|@distinctUnionOfObjects|NSArray|

例子：在不使用NSSet的情况下将数组中的重复元素去除掉
```
[array valueForKeyPath: @"@distinctUnionOfObjects.self"]
```

3. 集合数组操作

|符号|返回类型|
|:---:|:---:|
|@unionOfArrays|NSArray or NSSet|
|@distinctUnionOfArrays|NSArray or NSSet|
|@distinctUnionOfSets|NSArray or NSSet|

* NSDataDetector可以帮助我们从文本中获取某些指定类型的信息，目前支持的信息类型有：日期、地址、链接、电话号码和航旅信息等。

使用范例：

```
NSString *string = @"123 Main St. / (555) 555-5555";

NSError *error;
NSDataDetector *detector = [NSDataDetector dataDetectorWithTypes: NSTextCheckingTypeLink | NSTextCheckingTypePhoneNumbere error: &error];
[detector enumerateMatchesInString: string options: kNilOptions range: NSMakeRang(0, [string length])
            usingBlock:^(NSTectCheckingResult *result, NSMatcihngFlagsf flags, BOOL *stop) {
                NSLog(@"Match: %@", result);
            }
}];
```











