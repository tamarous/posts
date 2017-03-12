# Masonry 源代码剖析
[Masonry](https://github.com/SnapKit/Masonry) 是一个用来代替苹果原生的AutoLayout 的自动布局框架。这个库的代码量不是很多，而且使用也很简单方便，那么就让我们深入到这个库的内部，看看它是怎么实现的。

## 使用范例
下面是一个使用Masonry 的范例：
    
    [view mas_makeConstraints: ^(MASConstraintMaker *make) {
        make.top.equalTo(superview.mas_top).with.offset(padding.top);
        make.left.equalTo(superview.mas_left).with.offset(padding.left);
        make.bottom.equalTo(superview.mas_bottom).with.offset(-padding.bottom);
        make.right.equalTo(superview.mas_right).with.offset(-padding.right);
    }];
    
可以看到，view 与superview 之间的约束，是通过一个block 设置的。这个block 接受一个类型为MASConstraintMaker 的参数make，然后在block 函数体内通过make 来实现约束设置。make 看起来像是view 的一个替身，当我们写
    
    make.top.equalTo(superview.mas_top).with.offset(padding.top);
    
时，实际效果是

    view.top.equalTo(superview.mas_top).with.offset(padding.top);

## 内部实现

### mas_makeConstraints:

在Xcode 中，按住Command，在方法上点击鼠标左键就可以进入方法的内部实现。我们进入`mas_makeConstraints:`，可以看到这个方法的实现如下：

    - (NSArray *)mas_makeConstraints:(void(^)(MASConstraintMaker *))block       {
        self.translatesAutoresizingMaskIntoConstraints = NO;
        MASConstraintMaker *constraintMaker = [[MASConstraintMaker alloc] initWithView: self];
        block(constraintMaker);
        return [constraintMaker install];
    }
        
首先要将被设置的view 的`translatesAutoresizingMaskIntoConstraints`属性设置成NO，这样才可以成功添加约束。

然后，创建一个`MASConstraintMaker`的实例。block 接受这个实例，然后执行block 块内的内容。最后返回`[constraintMaker install]`的结果。

### MASConstraintMaker
 
####初始化
    // MASContraintMaker.m
    
    // MAS_VIEW 是UIView 的别名
    @property (nonatomic, weak) MAS_VIEW *view;
    
    - (id) initWithView:(MAS_VIEW *) view {
        self = [super init];
        if (! self) {
            return nil;
        }
        self.view = view;
        self.constraints = NSMutableArray.new;
        return self;
    }

在初始化方法中，`MASContraintMaker` 将要设置的视图赋值给它的属性view，因此 `MASContraintMaker` 对将要设置的视图拥有了一个弱引用。

#### block 的执行
block 中实际进行了约束的设置工作。

        make.top.equalTo(superview.mas_top).with.offset(padding.top);
        make.left.equalTo(superview.mas_left).with.offset(padding.left);
        make.bottom.equalTo(superview.mas_bottom).with.offset(-padding.bottom);
        make.right.equalTo(superview.mas_right).with.offset(-padding.right);

这个调用过程是链式的，即每次调用都是作用于`MASConstraint` 类型上，而本次调用的返回类型仍然是`MASConstraint` 类型。

##### right,left,bottom,top
right，left，bottom和top 实际上是`MASConstraintMaker`中的相应属性的getter 方法，以top 为例，其定义如下：
    
    - (MASConstraint *) top {
        return [self addConstraintWithLayoutAttribute: NSLayoutAttributeTop];
    }
    
这个方法中还涉及到一些其他方法：

    - (MASContraint *) addConstraintWithLayoutAttribute:(NSLayoutAttribute) layoutAttribute {
        return [self constraint: nil addConstraintWithLayoutAttribute: layoutAttribute];
    }

    - (MASConstraint *) constraint:(MASContraint *)constraint addConstraintWithLayoutAttribute:(NSLayoutAttribute) layoutAttribute {
        MASViewAttribute *viewAttribute = [[MASViewAttribute alloc] initWithView:self.view layoutAttribute:layoutAttribute]; // 1
        MASViewAttribute *newAttribute = [[MASViewAttribute alloc] initWithFirstViewAttribute:viewAttribute]; // 2
        if ([constraint isKindOfClass: MASViewConstraint.class]) {
            NSArray *children = @[constraint, newConstraint];
            MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren: children];
            compositeConstraint.delegate = self;
            [self constraint: constraint shouleBeReplaceWithConstraint: compositeConstraint];
            return compositeConstraint;
        }
        // 因为传入的constraint 为nil，所以直接进入下面的判断
        if (! constraint) {
            newConstraint.delegate = self;
            [self.constraints addObject: newConstraint]; // 3
        }
        return newConstraint;
    }
    
在上面这个方法中：
1. 根据传入的要设置的`NSLayoutAttribute` 生成对应的`MASViewAttribute`。
2. 根据`MASViewAttribute` 生成当前视图的新约束。
3. 将2中生成的新约束加入`MASContraintMaker` 的约束数组中。

##### equalTo, greaterThanOrEqualTo, lessThanOrEqualTo    

    - (MASConstraint * (^)(id attr))equalTo;
    - (MASConstraint * (^)(id attr))greaterThanOrEqualTo;
    - (MASConstraint * (^)(id attr))lessThanOrEqualTo; 

这三个方法的参数是id 类型的，因此它们既可以接受NSValue 类型的参数，如NSNumber，CGPoint，CGSize等，也可以接受`UIView` 类型的参数，还可以接受`MASViewConstraint` 类型的参数。
当接受NSValue 类型的参数时，如
    
    make.top.equalTo(@100)
表示当前视图的top与它的superview 的top相距100；
当接受`UIView` 类型的参数时，会和`UIView` 的对应属性做比较，如

    make.top.equalTo(secondView)
表示当前视图和secondView 的top是平齐的；
当接受`MASViewConstraint` 类型的参数时，如

    make.bottom.equalTo(secondView.mas_bottom)
表示当前视图和secondView 的bottom 是平齐的。

这个特性的实现原理会在稍后进行介绍。

以上方法虽然定义不同，但是内部实现相似，都是调用了`MASConstraint` 的`equalToWithRelation` 方法，但是传入了不同的参数：
    
    - (MASConstraint * (^)(id attr))equalTo 「
        return ^id(id attribute) {
            return self.equalToWithRelation(attribute, NSLayoutRelationEqual);
        };
    }
    - (MASConstraint * (^)(id attr))greaterThanOrEqualTo {
        return ^id(id attribute) {
            return self.equalToWithRelation(attribute, NSLayoutRelationGreaterThanOrEqual);
        };
    }
    - (MASConstraint * (^)(id attr)) lessThanOrEqualTo{
        return ^id(id attribute) {
            return self.equalToWithRelation(attribute,
            NSLayoutRelationLessThanOrEqual);
        };
    }
            
继续向下寻找`equalToWithRelation` 的定义。使用Command + 鼠标左键，我们发现这个方法在三个地方有定义：
    
    [MASConstraint(Abstract) equalToWithRelation];
    [MASViewConstraint equalToWithRelation];
    [MASCompositeConstraint equalToWithRelation];

`MASCompositeConstraint` 和`MASViewContraint` 是`MASConstraint` 的子类。在`MASConstraint` 中定义了很多抽象方法，都是需要在这两个子类中进行实现的。
> MASCompositeConstraint: A composite with a predefined array of children.

从定义和名称中可以看出，`MASCompositeConstraint` 是约束的组合。按照我的理解，它的作用可以看作是一颗树的根，这个根确定了哪些节点位于这棵树上，而树的子节点的类型都是`MASViewConstraint`。

`equalToWithRelation` 是一个抽象方法，在`MASContraint` 中没有实现，如果调用此方法就会抛出一个异常：

    - (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation { MASMethodNotImplemented(); }

    #define MASMethodNotImplemented() \
    @throw [NSException exceptionWithName:NSInternalInconsistencyException \
                                   reason:[NSString stringWithFormat:@"You must override %@ in a subclass.", NSStringFromSelector(_cmd)] \
                                 userInfo:nil]

`MASViewConstraint` 重载了此方法：
    
        - (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation {
    return ^id(id attribute, NSLayoutRelation relation) {
        if ([attribute isKindOfClass:NSArray.class]) { // 1
            NSAssert(!self.hasLayoutRelation, @"Redefinition of constraint relation");
            NSMutableArray *children = NSMutableArray.new;
            for (id attr in attribute) {
                MASViewConstraint *viewConstraint = [self copy];
                viewConstraint.layoutRelation = relation;
                viewConstraint.secondViewAttribute = attr;
                [children addObject:viewConstraint];
            }
            MASCompositeConstraint *compositeConstraint = [[MASCompositeConstraint alloc] initWithChildren:children];
            compositeConstraint.delegate = self.delegate;
            [self.delegate constraint:self shouldBeReplacedWithConstraint:compositeConstraint];
            return compositeConstraint;
        } else {
            NSAssert(!self.hasLayoutRelation || self.layoutRelation == relation && [attribute isKindOfClass:NSValue.class], @"Redefinition of constraint relation");
            self.layoutRelation = relation;
            self.secondViewAttribute = attribute;  // 2
            return self;
        }
    };
    }
    
1. 判断传入的attribute 是否为NSArray 类型。
2. 如果不是，则将传入的参数赋值给某些属性。注意secondViewAttribute 这个属性的setter 方法，它并不是一个简单的赋值过程，而是会根据传入的attribute 的类型进行额外的设置。

        - (void)setSecondViewAttribute:(id)secondViewAttribute {

            if ([secondViewAttribute isKindOfClass:NSValue.class]) {
            
            // 1 对应于上文中的equalTo(@100) 这种情况
                [self setLayoutConstantWithValue:secondViewAttribute]; 
            } else if ([secondViewAttribute isKindOfClass:MAS_VIEW.class]) {
            
            // 2 对应于上文中的equalTo(secondView) 这种情况
                _secondViewAttribute = [[MASViewAttribute alloc] initWithView:secondViewAttribute layoutAttribute:self.firstViewAttribute.layoutAttribute]; // 2
            } else if ([secondViewAttribute isKindOfClass:MASViewAttribute.class]) {
            
            // 3 对应于上文中的equalTo(secondView.mas_bottom) 这种情况
                _secondViewAttribute = secondViewAttribute;
            } else {
                NSAssert(NO, @"attempting to add unsupported attribute: %@", secondViewAttribute);
            }
        }

`MASCompositeConstraint` 对此方法的实现则要简单得多：

    - (MASConstraint * (^)(id, NSLayoutRelation))equalToWithRelation {
        return ^id(id attr, NSLayoutRelation relation) {
            for (MASConstraint *constraint in self.childConstraints.copy) {
                constraint.equalToWithRelation(attr, relation);
        }
            return self;
        };
    }
    
可以看出，这个方法的实现是递归的，它对当前`MASCompositeConstraint` 的每一个childConstraint 调用对应的`equalToWithRelation` 方法，如果当前某一个constraint 的childConstraint 为空，就返回这个constraint。怎么样，是不是有点类似于后续遍历？
因为实际的工作都是由`MASViewConstraint` 类型完成的，下面我们就只关心方法在这个类中的实现。


#### 安装约束

    - (NSArray *)install {
        if (self.removeExisting) {
            NSArray *installedConstraints = [MASViewConstraint installedConstraintsForView: self.view]; // 1
            for (MASConstraint *constraint in installedConstraints) {
                [constraint uninstall]; // 2
            }
        }
        NSArray *constraints = self.constraints.copy;
        for (MASConstraint *constraint in constraints) {
            constraint.updateExisting = self.updateExisting;
            [constraint install]; // 3
        }
        [self.constraints removeAllObjects]; // 4
        return constraints;
    }
    

在`MASConstraintMaker`的`install`方法中，有如下几个过程：
1. 查看当前视图上已经安装的约束。
2. 对于**已经**安装的约束，逐一进行卸载。
3. 对于**将要**安装的约束，逐一`[constaint install]`，进行安装。
4. 将自身的`constraints`数组中的所有元素清除掉。

### MASViewConstraint
我们着重来看上一节描述的`[constraintMaker install]`中的第三点。


    // MASViewConstraint.m
    - (void)install {
    if (self.hasBeenInstalled) {
        return;
    }
    
    // 1
    if ([self supportsActiveProperty] && self.layoutConstraint) {
        self.layoutConstraint.active = YES;
        [self.firstViewAttribute.view.mas_installedConstraints addObject:self];
        return;
    }
    
    // 2
    MAS_VIEW *firstLayoutItem = self.firstViewAttribute.item;
    NSLayoutAttribute firstLayoutAttribute = self.firstViewAttribute.layoutAttribute;
    MAS_VIEW *secondLayoutItem = self.secondViewAttribute.item;
    NSLayoutAttribute secondLayoutAttribute = self.secondViewAttribute.layoutAttribute;

    // 3
    // alignment attributes must have a secondViewAttribute
    // therefore we assume that is refering to superview
    // eg make.left.equalTo(@10)
    if (!self.firstViewAttribute.isSizeAttribute && !self.secondViewAttribute) {
        secondLayoutItem = self.firstViewAttribute.view.superview;
        secondLayoutAttribute = firstLayoutAttribute;
    }
    
    // 4
    MASLayoutConstraint *layoutConstraint
        = [MASLayoutConstraint constraintWithItem:firstLayoutItem
                                        attribute:firstLayoutAttribute
                                        relatedBy:self.layoutRelation
                                           toItem:secondLayoutItem
                                        attribute:secondLayoutAttribute
                                       multiplier:self.layoutMultiplier
                                         constant:self.layoutConstant];
    
    layoutConstraint.priority = self.layoutPriority;
    layoutConstraint.mas_key = self.mas_key;
    
    
    // 5
    if (self.secondViewAttribute.view) {
        MAS_VIEW *closestCommonSuperview = [self.firstViewAttribute.view mas_closestCommonSuperview:self.secondViewAttribute.view];
        NSAssert(closestCommonSuperview,
                 @"couldn't find a common superview for %@ and %@",
                 self.firstViewAttribute.view, self.secondViewAttribute.view);
        self.installedView = closestCommonSuperview;
    } else if (self.firstViewAttribute.isSizeAttribute) {
        self.installedView = self.firstViewAttribute.view;
    } else {
        self.installedView = self.firstViewAttribute.view.superview;
    }


    MASLayoutConstraint *existingConstraint = nil;
    if (self.updateExisting) {
        existingConstraint = [self layoutConstraintSimilarTo:layoutConstraint];
    }
    if (existingConstraint) {
        // just update the constant
        existingConstraint.constant = layoutConstraint.constant;
        self.layoutConstraint = existingConstraint;
    } else {
        [self.installedView addConstraint:layoutConstraint];
        self.layoutConstraint = layoutConstraint;
        [firstLayoutItem.mas_installedConstraints addObject:self];
    }
    }



在对代码进行分析之前，我们需要知道：因为每个约束需要处理的，是两个item 之间的关系。
>item1.attribute1 = multiplier × item2.attribute2 + constant // 约束等式

所以`MASViewConstraint` 有两个属性，分别是
    
    @property (nonatomic, strong, readonly) MASViewAttribute *firstViewAttribute;

和
    
    @property (nonatomic, strong, readonly) MASViewAttribute *secondViewAttribute;

这两个属性分别描述了约束等式的左边和右边。
因此，代码的执行过程为：

1. 将当前约束加入约束等式左边的view 的已安装约束列表中。
2. 获取当前约束的左右两边的item 和它们对应的layoutAttribute。
3. 如果item1 描述的不是数值，并且item2 为nil，则当前约束对应着前面所说的`equalTo(@100)`这种情况，所以当前的约束的item2 应该是item1中view 的superview。这和我们平时手写UI时的逻辑是相同的。
4. 调用系统API 来进行约束设置。
5. 如果item2 是一个view，那么这个约束应该被安装在item1 和item2 的共同superview 上，因此需要找到item1 和item2 的最近共同superview。其他情况同理。
    
```
- (instancetype)mas_closestCommonSuperview:(MAS_VIEW *)view {
    MAS_VIEW *closestCommonSuperview = nil;

    MAS_VIEW *secondViewSuperview = view;
    while (!closestCommonSuperview && secondViewSuperview) {
        MAS_VIEW *firstViewSuperview = self;
        while (!closestCommonSuperview && firstViewSuperview) {
            if (secondViewSuperview == firstViewSuperview) {
                closestCommonSuperview = secondViewSuperview;
            }
            firstViewSuperview = firstViewSuperview.superview;
        }
        secondViewSuperview = secondViewSuperview.superview;
    }
    return closestCommonSuperview;
}
```

## 小结
本文简要分析了用`Masonry` 设置视图间布局约束时的代码的内部实现。从分析过程中，我感受到了写一个库的难度，也体会到了那句话的含义：“将简洁留给用户，将复杂留给自己”。无论是`SDWebImage` 的一行代码设置图片还是`Masonry` 的链式设置方法，其简洁的背后藏有复杂的逻辑，恼人的边界处理和多种策略的应用。






