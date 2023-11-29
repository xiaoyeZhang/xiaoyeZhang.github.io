---
title: iOS - weak属性
---

# 一、简介

ARC 是自 iOS 5 之后增加的新特性，完全消除了手动管理内存的烦琐，编译器会自动在适当的地方插入适当的 retain、release、autorelease 语句。你不再需要担心内存管理,因为编译器为你处理了一切.

注意：ARC 是编译器特性，而不是 iOS 运行时特性(除了 weak 指针系统)，它也不是类似于其它语言中的垃圾回收器。因此 ARC 和手动内存管理性能是一样的，有时还能更加快速，因为编译器还可以执行某些优化。

# 二、weak 的作用

iOS 开发程序猿都会知晓 weak 关键字的作用是弱引用，所引用对象的计数器不会加一，并在引用对象被释放的时候自动被设置为 nil。而如果大量使用附有 weak 修饰符的变量，则会消耗相应的 CPU 资源。所以我们只需要避免循环引用时使用\_ \_ weak 修饰符。

# 三、weak 实现原理

Runtime 维护了一个 weak 表，用于存储指向某个对象的所有 weak 指针。weak 表其实是一个 hash（哈希）表，Key 是所指对象的地址，Value 是 weak 指针的地址（这个地址的值是所指对象的地址）数组。

weak 的实现原理可以概括以下三步：

1、初始化时：runtime 会调用 objc_initWeak 函数，初始化一个新的 weak 指针指向对象的地址。 2、添加引用时：objc_initWeak 函数会调用 objc_storeWeak() 函数， objc_storeWeak() 的作用是更新指针指向，创建对应的弱引用表。 3、释放时，调用 clearDeallocating 函数。clearDeallocating 函数首先根据对象地址获取所有 weak 指针地址的数组，然后遍历这个数组把其中的数据设为 nil，最后把这个 entry 从 weak 表中删除，最后清理对象的记录。

# 章后彩蛋

- 实现 weak 后，为什么对象释放后会自动为 nil？ runtime 对注册的类， 会进行布局，对于 weak 对象会放入一个 hash 表中。 用 weak 指向的对象内存地址作为 key，当此对象的引用计数为 0 的时候会 dealloc，假如 weak 指向的对象内存地址是 a ，那么就会以 a 为键， 在这个 weak 表中搜索，找到所有以 a 为键的 weak 对象，从而设置为 nil 。

- 当 weak 引用指向的对象被释放时，又是如何去处理 weak 指针的呢？ 1、调用 objc_release 2、因为对象的引用计数为 0，所以执行 dealloc 3、在 dealloc 中，调用了\_objc_rootDealloc 函数 4、在\_objc_rootDealloc 中，调用了 object_dispose 函数 5、调用 objc_destructInstance 6、最后调用 objc_clear_deallocating(重点关注),详细过程如下：

```
objc_clear_deallocating该函数的动作如下：

1、从weak表中获取废弃对象的地址为键值的记录

2、将包含在记录中的所有附有 weak修饰符变量的地址，赋值为nil

3、将weak表中该记录删除

4、从引用计数表中删除废弃对象的地址为键值的记录

```

[iOS 底层解析 weak 的实现原理（包含 weak 对象的初始化，引用，释放的分析）](http://www.cocoachina.com/ios/20170328/18962.html)
