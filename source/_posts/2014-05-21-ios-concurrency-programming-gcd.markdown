---
layout: post
title: "iOS 并发编程之 GCD"
date: 2014-05-21 11:40:27 +0800
comments: true
categories: [Archives, iOS Development]
keywords: GCD，iOS, Dispatch Queue, Concurrency Programming
description: iOS 并发编程之 GCD
---

在 iOS 并发编程之 Operation 中我们提到了 GCD 出现的背景，这篇文章是我对它的使用总结。

##GCD 是什么？  
> Grand Central Dispatch (GCD) dispatch queues are a powerful tool for performing tasks. Dispatch queues let you execute arbitrary blocks of code either asynchronously or synchronously with respect to the caller.

GCD 分发队列是执行任务的强大工具。 分发队列可以让你异步或同步运行任务代码块。  

##为什么使用 GCD?
我们使用 GCD 的原因很可能是我们想要异步或同步运行执行任务，并且还想获得以下优势:  

* 直接简单的编程接口；  
* 自动、整体的线程池管理；  
* 高效的内存管理；  
* 负载时不干扰内核；  
* 异步分发任务到分派队列不会造成死锁；  
* 优雅地处理竞争；  
* 串行分派队列为锁和其他同步操作提供了更高效选择；  

##如何使用 GCD？
在 GCD 是什么部分，我们指出它是执行任务的分发队列。除了核心的分发队列，GCD 还提供了几个其他的使用分发队列的技术来帮助我们管理代码。

###Dispatch groups

Dispatch group 

Dispatch group 是一种监视一系列块对象已完成的方法。(你可以根据需求同步或异步地监视块。)它为需要依赖其他任务完成的代码提供了有用的同步机制。

###Dispatch semaphores
Dispatch semaphores 类似传统的信号量，但是通常更加高效。它仅仅在信号量不可用需要阻塞线程时才向下调用到内核。如果信号量可用，无需内核调用。

###Dispatch sources
Dispatch source 产生通知响应指定的系统事件。你可以使用 dispatch sources 来监视像进程通知，信号和描述符等类似事件。当事件发生时，dispatch source 异步地提交你的任务到指定分发队列去处理。

所以要掌握如何使用 GCD，我们需要学习如何使用 Dispatch queue, Dispatch groups, Dispatch semaphores 和 Dispatch sources。

<!--more-->
##Dispatch Queue
###关于分发队列
分发队列是类对象的结构，它管理你提交的任务。所有分发队列都是先进先出。因此，任务开始的顺序就是你提交的顺序。 GCD 自动为你提供了一些分发队列，你也可以根据具体的需求创建其他的队列。表 3-1 列出了你应用可用分发队列的类型以及如何使用它们。  

表 3-1 分发队列的类型

| 类型 | 描述 |
| :---: | :---- |
| Serial | 串行队列(也称作私有分发队列)按任务添加的顺序一次执行一个任务。当前运行的任务跑在队列管理确定的线程上。串行队列通常用来同步访问特定资源。<br>你可以创建任意多的串行队列，它们相互之间是并行的关系。换句话说，如果你创建了四个中午队列，每个一次执行一个任务，但是分属四个队列的四个任务却是并行的。 |
| Concurrent | 并行队列(也称作一种类型的全局分发队列)并发地执行一个或多个任务，但是任务开始的顺序仍然是按它们加入队列的顺序。并发执行的任务跑在分发队列管理确定的线程上。给点任意点准确运行的余力数量是可变的并且依赖系统条件。<br>在 iOS 5 及以上，你自己可以通过指定 DISPATCH_QUEUE_CONCURRENT 作为队列类型创建并发分发队列。此外，这里还有四个预先定义的全局并发队列供你的应用使用。 |
| Main dispatch queue | 主分发队列是全局可用的串行队列，它执行应用主线程上的任务。此队列使用应用程序的运行循环 (如果存在), 将排队任务的执行与附加到运行循环的其他事件源进行交错。因为它运行在你应用的主线程上，主队列经常用作应用的关键同步点。<br>虽然你不需要创建主分发队列，但你需确保你的应用合适地耗尽它。 |

我们使用 GCD 的方法就是往合适的队列里提交任务，而获得合适队列的方法从上面的介绍我们知道是创建或获取。

###创建和获取队列

```
// 1.Serial queue
dispatch_queue_t myCustomSerialQueue = dispatch_queue_create("com.example.MyCustomSerialQueue", DISPATCH_QUEUE_SERIAL);

// 2.Concurrent queue
// Get concurrent queue
dispatch_queue defaultGlobalConcurrentQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

// Create concurrent queue
dispatch_queue_t myCustomConcurrentQueue = dispatch_queue_create("myCustomConcurrentQueue", DISPATCH_QUEUE_CONCURRENT);

// 3. Main queue
dispatch_queue_t mainQueue = dispatch_get_main_queue(void);
```

###用块实现任务

在 GCD 中，任务是用块来实现的，例如：

```
int x = 123;
int y = 456;

// Block declaration and assignment

void (^aBlock)(int) = ^(int z) {

    printf("%d %d %d\n", x, y, z);

};

// Execute the block

aBlock(789);   // prints: 123 456 789

```

###添加任务到队列

有两种方法添加任务到队列：异步或者同步。只要可能，使用 dispatch_async 和 dispatch_async_f 函数异步运行是优于同步的。当你添加块或函数到队列，没办法知道它们何时将被执行。因此，异步地添加块对象或函数让你从调用线程调度代码的运行，然后继续做其他工作。如果你从应用的主线程来调度任务，这尤为重要。

虽然你应该应该尽可能地异步添加任务，你仍然会有需要同步添加任务去防止竞争条件或其他同步错误的时候。在这些时候，你可以使用 dispatch_sync 和 dispatch_sync_f 添加任务到队列。这些函数会阻塞当前线程的执行直到指定的任务完成。

```
dispatch_queue_t myCustomQueue;

myCustomQueue = dispatch_queue_create("com.example.MyCustomQueue", NULL);

dispatch_async(myCustomQueue, ^{
    printf("Do some work here.\n");
});

printf("The first block may or may not have run.\n");

dispatch_sync(myCustomQueue, ^{
    printf("Do some more work here.\n");
});

printf("Both blocks have completed.\n");


```

##Dispatch Groups

##Dispatch Semaphores

##Dispatch Sources

##Reference
o Concurrency Programming Guide    
o [Grand Central Dispatch In-Depth: Part 1/2](http://www.raywenderlich.com/60749/grand-central-dispatch-in-depth-part-1)   
o [Grand Central Dispatch In-Depth: Part 2/2](http://www.raywenderlich.com/63338/grand-central-dispatch-in-depth-part-2)  
o [iOS多线程编程Part 3/3 - GCD](http://www.hrchen.com/2013/07/multi-threading-programming-of-ios-part-3/)

