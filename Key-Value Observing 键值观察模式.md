# Key-Value Observing 键值观察模式

键值观察模式（下面简称KVO）是建立在 KVC 的基础上的，关于 KVC，在前面[一篇文章](http://www.tamarous.com/2016/09/21/key-value-coding-%E9%94%AE%E5%80%BC%E7%BC%96%E7%A0%81/)中已有介绍。

## 如何使用KVO
如果要使用 KVO 模式，那么需要进行以下几步：

1. 使用 `addObserver:forKeyPath:options:context:` 来向一个被观察的对象注册观察者。
2. 在观察者内部实现 `observeValueForKeyPath:ofObject:change:context` 来接收被观察对象发生改变时发出的通知。
3. 当不需要接收通知时，实现 `removeObserver:forKeyPath:` 来取消对该观察者的注册。

### 注册为观察者

一个对象通过向自己发送 `addObserver:forKeyPath:options:context:` 消息，来成为待观察对象的观察者。这四个参数，第一个通常传入self，第二个为待观察对象的 keypath，第三个为观察选项，而第四个为上下文，通常用于判断当前通知由哪一个待观察对象发出，下文中将详细说明。

#### 选项

选项这个参数，由一些选项常数进行或运算后形成。它不仅影响通知中提供的 change 词典，也会影响通知的产生方式。

常见的选项常数：

* NSKeyValueObservingOptionOld. 这个选项可以让你在change词典中获得被观察对象改变前的值。
* NSKeyValueObservingOptionNew. 这个选项可以让你在change词典中获得被观察对象改变后的新值。
* NSKeyValueObseringOptionInitial. 这个选项可以让你在change词典中获得被观察对象的初始值。
* NSKeyValueObservingOptionPrior. 这个选项可以让被观察者在被观察值发生之前发出通知告知观察者。

如果使用 NSKeyValueObservingOptionOld | NSKeyValueObservingOptionNew，那么就可以在 change 词典中同时获得被观察对象改变前和改变后的值。

#### 上下文
按照我的理解，上下文这个参数的作用是用来确保被观察对象发生改变时发出的通知被发送到正确的观察者手上。通常先声明一个静态变量，然后将这个静态变量的地址传入 context 中，如：

    static void *PersonAccountBalanceContext = &PersonAccountBalanceContext;
    static void *PersonAccountInterestRateContext = &PersonAccountInterestRateContext;
    
    - (void) registerAsObserverForAccount:(Account *) account {
        [account addObserver:self forKeyPath:@"balance" options: (NSKeyValueObservingOptionNew | NSKeyObservingOptionOld) context: PersonAccountBalanceContext];
        
        [account addObserver:self forKeyPath:@"interest" options: (NSKeyValueObservingOptionNew | NSKeyObservingOptionOld) context: PersonAccountInterestContext];
    
    }
    
    
#### 在发生变化时接收通知

当某个对象的被观察属性的值发生变化时，观察者会收到一个`observeValueForKeyPath:ofObject:change:context:`消息。所有的观察者都必须实现这个方法。此方法中的 change，是一个 NSDictionary，这个词典中存储了被观察属性在变化前后的值，可以通过键的方式来取出这些值。

例如，NSKeyValueChangeOldKey 可以取出发生改变前的属性值，NSKeyValueChangeNewKey 可以取出发生改变后的属性值。如果被观察对象发生了改变，那么 NSKeyValueChangeKindKey 会取出 NSKeyValueChangeSetting。

如果被观察对象是一个一对多关系，那么通过 NSKeyValueChangeKindKey 能够获知被观察对象中是否插入(返回 NSKeyValueChangeInsertion)、删除(返回 NSKeyValueChangeRemoval)、替换(返回 NSKeyValueChangeReplacement)了某些值。

    - (void)observeValueForKeyPath:(NSString *)keyPath
                      ofObject:(id)object
                        change:(NSDictionary *)change
                       context:(void *)context {
 
        if (context == PersonAccountBalanceContext) {
            // Do something with the balance…
     
        } else if (context == PersonAccountInterestRateContext) {
            // Do something with the interest rate…
     
        } else {
            // Any unrecognized context must belong to super
            [super observeValueForKeyPath:keyPath
                                 ofObject:object
                                   change:change
                                   context:context];
        }
    }


如果说观察者不能识别 context ，那么它应该在 `observeValueForKeyPath:ofObject:change:context` 中调用超类的此方法。

**注意**：如果说通知被分发到了类层次的顶端，那么 NSObject 会抛出一个 `NSInternalInconsistencyException`。

### 移除注册

通过向观察者发送 `removeObserver:forKeyPath:context` 消息，可以移除某个观察者。

    - (void)unregisterAsObserverForAccount:(Account*)account {
        [account removeObserver:self
                     forKeyPath:@"balance"
                        context:PersonAccountBalanceContext];
     
        [account removeObserver:self
                     forKeyPath:@"interestRate"
                        context:PersonAccountInterestRateContext];
    }

当移除一个观察者时，记住以下几个要点：

* 如果一个取消一个未注册的观察者，那么会导致一个 NSRangeException 异常。避免这个问题的办法是每调用一次`addObserver:forKeyPath:options:context:`，就对应调用一次`removeObserver:forKeyPath:context:`。另外也可以将移除观察者的代码放在 try/catch 中。
* 当观察者被释放时，它不会自动调用remove方法，但是通知仍然会被发送给这个观察者。如果说一个被释放的对象收到一条消息，就会引发内存访问异常。因此在释放观察者时，必须记得要调用remove方法。
* 没有任何手段可以知道一个对象是否是一个观察者或者被观察者。因此，必须手动避免上面提到的问题。一个典型的做法是在观察者的初始化方法中将它注册为观察者，而在它的dealloc方法中移除。

## 遵循 KVO 规范

对于某一个特定的属性，如果它需要遵循 KVO 规范，那么这个类应该：

* 遵循 KVC 规范
* 会为属性的变化发送通知
* 相应的键应该被恰当地注册

有两种技术可以确保当属性值变化时会发送通知。一种是由 NSObject 提供的，即如果一个类的所有属性都遵循 KVC 时，那么它的所有属性都会自动支持这个特性。

另一种技术是手动地发送通知。这种方法提供了对通知发送时机的额外控制能力，因此需要一些额外的代码。 在类方法 `automaticallyNotifiesObserversForKey:` 中，对于想要手动管理通知的属性，让此方法返回NO，其他想要自动通知的属性则调用超类的此方法。

    + (BOOL) automaticallyNotifiesObserversForKey:(NSString *) theKey {
        
        BOOL automatic = NO;
        if ([theKey isEqualToKey:@"balance"]) {
            automatic = NO;
        } else {
            automatic = [super automaticallyNotifiesObserversForKey:theKey];
        }
        return automatic;
    }

### 手动发送通知

在手动发送通知中，当属性值发生改变之前，调用 `willChangeValueForKey:`，在属性值发送改变之后，调用 `didChangeValueForKey:`。

    - (void) setBalance:(double) theBalance {
        [self willChangeValueForKey:@"balance"];
        _balance = theBalance;
        [self didChangeValueForKey:@"balance"];
    }

如果在发送通知前先检查下属性值是否发生了更改，那么可以避免无用的通知。

    - (void) setBalance:(double) theBalance {
        if (theBalance != _balance) {
        
            [self willChangeValueForKey:@"balance"];
            _balance = theBalance;
            [self didChangeValueForKey:@"balance"];
        }
    }


如果某一个操作导致多个键改变，那么必须将通知嵌套在一起。

    - (void)setBalance:(double)theBalance {
        [self willChangeValueForKey:@"balance"];
        [self willChangeValueForKey:@"itemChanged"];
        _balance = theBalance;
        _itemChanged = _itemChanged+1;
        [self didChangeValueForKey:@"itemChanged"];
        [self didChangeValueForKey:@"balance"];
    }
    
在一个有序的一对多关系中，不仅需要指定发生变化的键，同时也要指出变化的类型和受影响的索引。变化的类型的键是 NSKeyValueChange，值则是
NSKeyValueChangeInsertion，NSKeyValueChangeRemoval，或 NSKeyValueChangeReplacement。受影响的索引则是一个 NSIndexSet对象。

    - (void)removeTransactionsAtIndexes:(NSIndexSet *)indexes {
        [self willChange:NSKeyValueChangeRemoval
            valuesAtIndexes:indexes forKey:@"transactions"];
     
        // Remove the transaction objects at the specified indexes.
     
        [self didChange:NSKeyValueChangeRemoval
            valuesAtIndexes:indexes forKey:@"transactions"];
    }


## 注册相互影响的键

### 一对一关系
举例来说，人的fullName由firstName和lastName组成。获得全名的方法可以这样写：

    - (NSString *) fullName {
        return [NSString stringWithFormat:@"%@ %@",firstName,lastName];
    }
    
 假设一个观察者观察fullName这个属性，那么当firstName和lastName发生变化时，它也应该收到通知。
 
 一种方法是重载 keyPathsForValueAffectingValuesForKey:，在这个方法中声明fullName属性依赖于lastName和firstName两个属性。
 
    + (NSSet *) keyPathsForValueAffectingValueForKey: (NSString *) key {
        NSSet *keyPaths = [super keyPathsForValuesAffectingValueForKey:key];
        if ([key isEqualToString:@"fullName"]) {
            NSArray \*affectingKeys = @[@"lastName",@"firstName"];
            keyPaths = [keyPaths setByAddingObjectsFromArray:affectingKeys];
        }
        return keyPaths;
    } 
    
第二种方法是实现一个遵循 keyPathsForValuesAffecting<Key>命名传统的类方法，Key是依赖于其他属性的那个attribute的名字(注意首字母要大写)。

    + (NSSet *) keyPathsForValuesAffectingFullName {
        return [NSSet setWithObjects:@"lastName",@"firstName",nil];
    }
    
### 一对多关系

上面的方法不支持一对多关系的keypaths。举例来说，有一个Department对象，这个对象中有一个employees数组，每个元素是一个Employee类，而Employee类有salary属性。假设你需要观察Department的totalSalary属性，这个属性是每一个Employee的salary之和。
那么实现keyPathsForValuesAffectingTotalSalary方法并不能收到通知。

有两种方法来实现需要的效果：

1. 使用 KVO 来将parent 注册为children相关属性的观察者。当将 child 添加和移除出 parent 时，也要同时将parent 注册和取消注册为child 的观察者。

        - (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
         
            if (context == totalSalaryContext) {
                [self updateTotalSalary];
            }
            else
            // deal with other observations and/or invoke super...
        }
         
        - (void)updateTotalSalary {
            [self setTotalSalary:[self valueForKeyPath:@"employees.@sum.salary"]];
        }
         
        - (void)setTotalSalary:(NSNumber *)newTotalSalary {
         
            if (totalSalary != newTotalSalary) {
                [self willChangeValueForKey:@"totalSalary"];
                _totalSalary = newTotalSalary;
                [self didChangeValueForKey:@"totalSalary"];
            }
        }
         
        - (NSNumber *)totalSalary {
            return _totalSalary;
        }

2. 如果使用 Core Data，那么可以将 parent在应用的 notification center 中注册为它管理的对象的观察者。parent 应该像 KVO 一样对 children 发出的通知做出反应。

