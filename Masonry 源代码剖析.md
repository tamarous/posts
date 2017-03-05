# Masonry 源代码剖析
[Masonry](https://github.com/SnapKit/Masonry)是一个用来代替苹果原生的AutoLayout 的自动布局框架。这个库的代码量不是很多，而且使用也很简单方便，那么就让我们深入到这个库的内部，看看它是怎么实现的。

## 使用范例
Masonry 开发团队给出了一个使用Masonry 的范例：
    
    [view mas_makeConstraints: ^(MASConstraintMaker *make) {
        make.top.equalTo(superview.mas_top).with.offset(padding.top);
        make.left.equalTo(superview.mas_left).with.offset(padding.left);
        make.bottom.equalTo(superview.mas_bottom).with.offset(-padding.bottom);
        make.right.equalTo(superview.mas_right).with.offset(-padding.right);
    }];
    
可以看到，view 与superview 之间的约束，是通过一个block 设置的。这个block 接受一个类型为MASConstraintMaker 的参数make，然后在block 函数体内通过make 来实现约束设置。


        



