---
title: iOS 90%人可能都会回答错误的一个多线程基础题
date: 2020-03-27 17:55:23
tags:
---
题目描述：

一问：GCD是同步还是异步情况会开启多线程

答：同步是不会开启新的线程的，异步才会开启新的线程。

这个没啥难度，基本都是必会的。通过代码验证同步在串行队列和并发队列情况下会不会创建新的线程

验证代码：
```swift
    dispatch_queue_t serialQueue = dispatch_queue_create("serialQueue", DISPATCH_QUEUE_SERIAL);
    dispatch_queue_t conQueue = dispatch_queue_create("conQueue", DISPATCH_QUEUE_CONCURRENT);

    NSLog(@"(1).=====%@",[NSThread currentThread]);
    dispatch_sync(serialQueue, ^{
      NSLog(@"(2).=====%@",[NSThread currentThread]);
    });
    dispatch_sync(conQueue, ^{
      NSLog(@"(3).=====%@",[NSThread currentThread]);
    });
```

输出结果：
```swift
(1).=====<NSThread: 0x2837f6f00>{number = 1, name = main}
(2).=====<NSThread: 0x2837f6f00>{number = 1, name = main}
(3).=====<NSThread: 0x2837f6f00>{number = 1, name = main}
```
可以看出同步是不会产生新的线程。当然问题肯定不会这么简单就结束了。

---

二问：异步一定会开启新的线程吗。

答：不会，异步在主队列里不会创建新的线程，在其他串行和并发队列都会创建新的子线程

验证代码：
```swift
    dispatch_queue_t mainQueue = dispatch_get_main_queue();
    dispatch_queue_t serialQueue = dispatch_queue_create("serialQueue", DISPATCH_QUEUE_SERIAL);
    dispatch_queue_t conQueue = dispatch_queue_create("conQueue", DISPATCH_QUEUE_CONCURRENT);

    NSLog(@"(1).=====%@",[NSThread currentThread]);
    dispatch_async(serialQueue, ^{
      NSLog(@"(2).=====%@",[NSThread currentThread]);
    });
    dispatch_async(conQueue, ^{
      NSLog(@"(3).=====%@",[NSThread currentThread]);
    });
    dispatch_async(mainQueue, ^{
      NSLog(@"(4).=====%@",[NSThread currentThread]);
    });
```
输出结果：
```swift
(1).=====<NSThread: 0x2800a6f00>{number = 1, name = main}
(2).=====<NSThread: 0x2800ca5c0>{number = 3, name = (null)}
(3).=====<NSThread: 0x2800ca5c0>{number = 3, name = (null)}
(4).=====<NSThread: 0x2800a6f00>{number = 1, name = main}
```
看结果(1)和(4)可以确定，在异步主队列确实没有开启新的线程，两个都是main线程，number号为1.

再看(2)和(3)，发现确实是创建了新的线程。但是仔细看线程号  **发现（2）和（3） 对应的 number 都是3** 。
也就是这两个异步动作只创建了一个新的线程。按照常识来说不是应该创建两个不同线程吗？

这儿就要从时间和空间谈到GCD对线程调度优化问题了。

GCD在执行多个异步操作的时候，会从时间和空间进行权衡会不会创建新的线程，一般每一个任务开启一个线程这样时间最优，但是线程太多会消耗内存空间。所以GCD会自动权衡根据任务分配合适的线程数，从而达到空间和时间的最优。

当然具体GCD是怎么做的，可能需要看源码了...

这是我给这个问题的总结：

* 同步：不具备开启线程的能力，一定串行执行任务    
* 异步：具有开启线程的能力，但是在主队列里不会开启新的线程。
如果在串行队列和并发队列里开启n个子线程，gcd优化之后未必会真的有n个子线程。


