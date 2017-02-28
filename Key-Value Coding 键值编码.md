# Key-Value Coding 键值编码

KVC和KVO是iOS开发中常见的两种技术，那么它们分别该如何使用，有哪些注意事项呢？在阅读了Apple的官方文档KVC Programming Guide和KVO Programming Guide后，将一些注意事项记录如下。

首先介绍下KVC。

下面是Apple官方文档中对Key-value coding（以下简称为KVC）的定义：

> Key-value coding is a mechanism for accessing an object’s properties indirectly, using strings to identify properties, rather than through invocation of an accessor method or accessing them directly through instance variables.

在应用中使用KVC是一条重要设计原则，因为它是KVO、Core Data、Cocoa Binding等技术的基础，同时也可以简化代码。

## KVC 基础

key是一个指明了一个对象的某一属性的字符串，而keypath则是将几个key用"."连接而成的一个长字符串。
### 使用KVC来设置和访问属性的值

假设有一个类MyClass，其声明如下:
    
    @interface MyClass
    @property NSString *stringProperty;
    @property NSInteger integerProperty;
    @property MyClass *linkedInstance;
    @end
    
那么设置属性时可以使用`setValue:forKey:`或者`setValue:forKeyPath:`方法，如

    MyClass *myInstance = [MyClass new];
    [myInstance setValue:@2 forKey:@"integerProperty"];
    [myInstance setValue:@3 forKeyPath:@"linkedInstance.integerProperty"];
    
除此以外，还可以使用`setValuesForKeysWithDictionary:`方法来一次性设置对象的所有属性。默认实现会对每一个键值对调用`setValue:forKey:`方法。

需要注意的是，在`setValue:forKey:`和`setValue:forKeyPath:`方法中，如果指定的key并不存在，那么消息的接收者就会收到一条`setValue:forUndefinedKey:`消息。这个方法的默认实现是引发一个NSUndefinedExpection异常，当然你也可以重载这个方法。

如果将一个非对象的属性（即一个标量）赋值为nil会怎样呢？在这种情况下，接收者会给自己发送一条`setNilValueForKey:`消息。这个方法的默认实现是引发了一个NSInvalidArgumentException异常。

访问属性的值可以使用`valueForKey:`和`valueForKeyPath:`方法：

    NSString *string = [myInstance valueForKey:@"stringProperty];
    NSInteger intergerProperty = [myInstance valueForKeyPath:@"linkedInstance.integerProperty"];
    
它和点语法的结果相同：

    NSString *string = myInstance.stringProperty;
    NSInteger integerProperty = myInstance.linkedInstance.integerProperty;
    

## KVC 访问器方法

### 常用的访问器模式

一般来说，某属性的访问器的格式是：**-\<key>**或者**-is\<key>**，而某属性的赋值器的格式是：**set\<key>**，其中key是属性的名字。如：

    - (BOOL) hidden {
    
    }
    
    - (BOOL) isHidden {

    }
    
    - (void) setHidden: (BOOL) flag {

        // do something
        return;
    }
    
因为hidden是非对象性的，所以如果它被设置了nil，那么就会触发`setNilValueForKey:`方法。我们可以这样实现这个方法：

    - (void) setNilValueForKey: (NSString *) theKey {

        if ([theKey isEqualToString:@"hidden"]) {
            [self setValue:@YES forKey:@"hidden"];
        }
        else {
            [super setNilValueForKey:theKey];
        }
    }
    
### 一对多关系的属性的访问器模式
对于一对多的属性，虽然也可以使用-\<key>和-set\<key>这样的访问器模式，但是出于提高性能的考虑，最好实现一些额外的访问器方法。

#### 有序的一对多关系
有序的一对多关系的通常体现是NSArray和NSMutableArray。

##### 不可变的有序的一对多关系的获得器方法
要支持对一个有序一对多关系的只读访问，需要实现下面几个方法：

* `-countOf<key>`。**必须**。类似于NSArray的count方法。
* `-objectIn<key>AtIndex:`和`-<key>AtIndexed:`方法。这两个方法中的任意一个是必须的。它们对应着NSArray的objectAtIndex:和ObjectsAtIndexes:方法。
* `-get<key>:range:`。实现这个方法是可选的，但是会带来额外的性能收益。它对应着NSArray的getObjects:range:方法。

例子如下：

    - (NSUInteger) countOfEmployees {
        return [self.employees count];
    }
    
    - (id) objectInEmployeesAtIndex:(NSUInteger) index {

        return [employees objectAtIndex:index];
    }
    
    - (NSArray *) employeesAtIndexes:(NSIndexSet *) indexes {
    
        return [self.employees objectsAtIndexes:indexes];
    }
    
    - (void) getEmployees: (Employee * __unsafe_unretained *) buffer range:(NSRange) inRange {
    
        [self.employees getObjects:buffer range:inRange];
    }
    

##### 可变的有序的一对多关系的获得器方法

对于一个可变的有序的一对多关系，为了使它符合KVC，需要实现以下方法：

* `-insertObject:in<key>AtIndex:`或者-`insert<key>:atIndexes:`。这两个方法中的至少一个需要被实现。它们类似于NSMutableArray中的insertObject:atIndex:和insertObjects:atIndexes:.
* `-removeObjectFrom<key>AtIndex:`或者`-remove<key>AtIndexes:`。这两个方法中的至少一个需要被实现。他们类似于NSMutableArray中的removeObjectAtIndex:和removeObjectsAtIndexes:。
* `-replaceObjectIn<key>AtIndex:withObject:`或者`-replace<key>AtIndexes:with<key>:`。**可选**。

例子如下：

    - (void) insertObject: (Employee *) employee inEmployeesAtIndex:(NSUInteger) index {

        [self.employees insertObject:employee atIndex:index];
        return;
    }
    
    - (void) insertEmployees:(NSArray *) employeeArray atIndexes: (NSIndexSet *)indexes {

        [self.employees insertObjects:employeeArray atIndexes:indexes];
        return;
    }
    
    - (void) removeObjectFromEmployeesAtIndex:(NSUInteger) index {

        [self.employees removeObjectAtIndex:index];
    }
    
    - (void) removeEmployeesAtIndexes: (NSIndexSet *) indexes {
    
        [self.employees removeObjectAtIndexes: indexes];
    }
    
    - (void)replaceObjectInEmployeesAtIndex:(NSUInteger)index
                             withObject:(id)anObject {
        [self.employees replaceObjectAtIndex:index withObject:anObject];
    }
 
    - (void)replaceEmployeesAtIndexes:(NSIndexSet *)indexes
                        withEmployees:(NSArray *)employeeArray {
     
        [self.employees replaceObjectsAtIndexes:indexes withObjects:employeeArray];
    }

    
#### 无序的一对多关系
无序的一对多关系的通常体现是NSSet和NSMutableSet。

##### 不可变的无序的一对多关系的获得器方法
要支持对一个无序一对多关系的只读访问，需要实现下面几个方法：

* `-countOf<key>`。**必须**。这个方法对应着NSSet的count方法。
* `-enumeratorOf<key>`。**必须**。对应着NSSet的objectEnumerator方法。
* `-memberOf<key>`。**必须**。对应着NSSet的member:方法。


        - (NSUInteger)countOfTransactions {
            return [self.transactions count];
        }
         
        - (NSEnumerator *)enumeratorOfTransactions {
            return [self.transactions objectEnumerator];
        }
         
        - (Transaction *)memberOfTransactions:(Transaction *)anObject {
            return [self.transactions member:anObject];
        }
        

##### 可变的有序的一对多关系的获得器方法
对于一个可变的无序的一对多关系，为了使它符合KVC，需要实现以下方法：

* `-add<key>Object:`或者`-add<key>:`。这两个方法中的至少一个需要被实现。它们类似于NSMutableSet的addObject:方法。
* `-remove<key>Object:`或者`-add<key>:`。这两个方法中的至少一个必须被实现。它们类似于NSMutableSet的removeObject:方法。
* `-intersect<key>:`方法。**可选的**。类似于NSSet的intersectSet:方法。

        - (void)addTransactionsObject:(Transaction *)anObject {
            [self.transactions addObject:anObject];
        }
         
        - (void)addTransactions:(NSSet *)manyObjects {
            [self.transactions unionSet:manyObjects];
        }
        - (void)removeTransactionsObject:(Transaction *)anObject {
            [self.transactions removeObject:anObject];
        }
         
        - (void)removeTransactions:(NSSet *)manyObjects {
            [self.transactions minusSet:manyObjects];
        }
                
        - (void)intersectTransactions:(NSSet *)otherObjects {
            return [self.transactions intersectSet:otherObjects];
        }
        
##键值验证

键值验证的目的是为了实现对传入值的测试，以决定是否需要接受此值，或者是用新值代替，或者是拒绝此值。

### 验证方法的命名传统

验证方法的命名格式是`validate<key>:error:`。例子如下：

    - (BOOL) validateName:(id *)ioValue error:(NSError * autoreleasing *)outError {
    
        // do something
        return ...;
    }
    
### 实现一个验证方法

验证方法接受两个参数：一个是需要验证的对象，一个是用来返回错误信息的NSError实例。

验证方法执行时可能有三种情况：
    
1. 对象是有效的，所以返回一个YES。
2. 对象是无效的，并且有效值无法被创建，因此返回NO。
3. 一个新的有效对象被创建。在将参数指向这个新的对象后，返回一个YES。

        -(BOOL)validateName:(id *)ioValue error:(NSError * __autoreleasing *)outError{
     
        // The name must not be nil, and must be at least two characters long.
        if ((*ioValue == nil) || ([(NSString *)*ioValue length] < 2)) {
                if (outError != NULL) {
                    NSString *errorString = NSLocalizedString(
                        @"A Person's name must be at least two characters long",
                        @"validation: Person, too short name error");
                    NSDictionary *userInfoDict = @{ NSLocalizedDescriptionKey : errorString };
                *outError = [[NSError alloc] initWithDomain:PERSON_ERROR_DOMAIN
                                                        code:PERSON_INVALID_NAME_CODE
                                                    userInfo:userInfoDict];
                }
                return NO;
            }
            return YES;
        }
    
    
### 调用验证方法

可以直接调用验证方法，也可以使用validateValue:forKey:error:来间接调用。这个方法的默认实现是在接收者的方法列表中寻找名字符合`validate<key>:error:`的验证方法。如果找到了这个方法，就会调用这个方法。如果没有找到符合这个名字的方法，那么`validateValue:forKey:error:`就会返回YES。

## 确保满足KVC

### 属性以及一对一关系
* 实现名为`-<key>`, `-is<key>`的方法，或者拥有名为`<key>`或`_<key>`的实例变量。
* 如果是可变的，那么实现`-set<key>:`方法。
* 在`-set<key>:`方法中不能进行验证。
* 如果可以的话，实现`-validate<key>:error:`方法。

### 有序的一对多关系
* 实现一个返回值为数组的名为`-<key>`的方法或者拥有一个名为`<key>`或`_<key>`的数组实例变量。
* 实现`-countOf<key>`，及`-objectIn<key>AtIndex:`和`-<key>AtIndexes:`中的一个。
* 可以选择实现`-get<key>:range`来提高性能。

对于可变的有序一对多关系，需要：

* 实现`-insertObject:in<key>AtIdex:`或`-insert<key>:atIndexes:`中的一个。
* 实现`-removeObjectFrom<key>AtIndex:`或`-remove<key>AtIndexes:`中的一个。
* 可以选择实现`-replaceObjectIn<key>AtIndex:withObject:`或`-replace<key>AtIndexes:with<key>:`中的一个来提高性能。

### 无序的一对多关系
* 实现一个返回值为集合的名为-<key>的方法或者拥有一个名为<key>或_<key>的集合实例变量。
* 实现`-countOf<key>`，`-enumerator<key>`和`-memberOf<key>`方法。

对于可变的无序一对多关系，需要：

* 实现`-add<key>Object:`或`-add<key>:`的一个或两个。
* 实现`-remove<key>Object:`或`-remove<key>:`的一个或两个。
* 可以选择实现`-intersect<key>:`和`-set<key>:`方法来提高性能。

## 集合操作符

集合操作符是一些特殊的keypath，它们可以被当作参数传到valueForKeyPath:方法中。

> @avg 
    
    NSNumber *transactionAverage = [transactions valueForKeyPath:@"@avg.amount"];

> @count

    NSNumber *transactionCount = [transactions valueForKeyPath:@"@count"];
    
> @max,@min
    
    NSNumber *transactionMax = [transactions valueForKeyPath:@"@max"];
    

> @sum

    NSNumber *amountSum = [transactions valueForKeyPath:@"@sum.amout"];
    
    
> @distinctUnionOfObjects 返回一个数组，数组元素中的重复值被去除。

    NSArray *payees = [transactions valueForKeyPath:@"distinctUnionOfObjects.payee"];
    
> @unionOfObjects返回一个数组，数组元素中的重复值不会被去除。

@distinctUnionOfArrays和@unionArrays以及@distinctUnionOfSets与前面几个类似，因此不再赘述。




    


    


    

