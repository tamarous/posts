使用对象时经常需要拷贝它。在OC中，此操作通过copy方法完成。如果要使自己写的类支持拷贝操作，那么就需要实现**NSCopying**协议，该协议只有一个方法：

    - (instancetype) copyWithZone:(NSZone *)zone

在以前，内存被分为不同的区(zone)，而对象会被创建在某个区中。但是现在每个程序只有一个默认区，所以现在在写这个方法时，仍然需要这个参数，但是不用关心zone究竟是什么了。

copy方法由NSObject实现，该方法只是以默认区为参数来调用"copyWithZone:"。当我们在调用copy方法时，实际上需要实现的是"copyWithZone:"方法。