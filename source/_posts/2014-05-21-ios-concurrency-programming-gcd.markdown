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

Dispatch groups 是一种阻塞线程直到一个或多个任务完成运行的方法。当所有指定任务没有完成是你不能继续的地方可以使用这种行为。例如，在分发多个任务去计划某些数据，你也许会使用一个组来等待这些任务，当他们完成时处理它们的结果。另外一种使用 dispatch groups 的方法是把它当作 thread join 的替代品。与其启动多个子线程然后联接它们，你可以添加相关任务到 dispatch groups 然后等待整个组。

下面是个等待异步任务的实例：

```
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

dispatch_group_t group = dispatch_group_create();

// Add a task to the group
dispatch_group_async(group, queue, ^{
   // Some asynchronous work
});

 
// Do some other work while the tasks execute.

 
// When you cannot make any more forward progress,

// wait on the group to block the current thread.
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);

// Release the group when it is no longer needed.
dispatch_release(group);

```

##Dispatch Semaphores

如果你提交到分发队列的任务是使用某些有限的资源，你也许想要使用 dispatch semaphore 来调度同时使用该资源的任务数量。Dispatch semaphore 和正常信号量的工作有一点区别。当资源可用时，获取一个 dispatch semaphore 比传统的系统信号量花费地时间更少。这是因为这种特殊的情况 Grand Central Dispatch 不会向下调用到内核。唯一需要调用到内核的情况是当资源不可用时，系统需要停住你的线程直到收到信号量。

使用 dispatch semaphore 的语义如下：

1. 当你创建信号量时(使用 dispatch_semaphore_create 函数)，你可以指定一个正整数来表示可用的资源数量。
2. 每个任务调用 dispatch_semaphore_wait 等待一个信号量。
3. 当等待调用返回，使用资源并执行你的工作。
4. 当你使用完资源，释放它并调用 dispatch_semaphore_signal 函数发送信号量。
	
有关这些步骤如何工作的示例，考虑系统文件描述符的使用。每个应用程序只有有限的文件描述符可供使用。如果你有一个任务，它处理大量的文件，你可能不想一次打开所有的文件以至于用光文件描述符。相反，你可以在你的文件处理代码中使用信号量来限制每次使用的文件描述符数量。你在你任务中使用的基本代码片断如下：

```
// Create the semaphore, specifying the initial pool size
dispatch_semaphore_t fd_sema = dispatch_semaphore_create(getdtablesize() / 2);

// Wait for a free file descriptor
dispatch_semaphore_wait(fd_sema, DISPATCH_TIME_FOREVER);

fd = open("/etc/services", O_RDONLY);

// Release the file descriptor when done
close(fd);

dispatch_semaphore_signal(fd_sema);

```

##Dispatch Sources

Dispatch source 是一个协调指定低层系统事件的数据类型。Grand Central Dispatch 支持如下类型的 dispatch source:

* Timer dispatch sources 产生周期性的通知。
* Signal dispatch sources 当一个 UNIX 信号来到时通知你。
* Descriptor sources 通知你各种基于文件和 socket 的操作，例如：
	
	* 当数据可读时
	* 可以写出数据时
	* 当文件被删除、移动或重命名时
	* 当文件的元数据信息改变时

* Process dispatch sources 通知你进程相关的事件，例如：

	* 当进程退出时
	* 当进程分发了一个 fork 或 exec 类型调用
	* 当一个信号到达了进程

* Mach port dispatch sources 通知你 Mach 相关的事件
* Custom dispatch sources 由你自己定义和触发

Dispatch sources 替换了异步回调函数，它过去被用于处理系统相关的事件。当你配置 dispatch source，你指定你想要监视的事件，分发队列和用来处理事件的代码。你可以使用 block 对象或者函数。当一个感兴趣的事件到达，dispatch source 提交你的 block 或函数到指定的分发队列。

和你手动提交到队列的任务不同，dispatch sources 为你的应用提交一个连续的事件源。dispatch source 一直保持附加到它自己的分发队列，除非你显示取消。当被附加后，任何时候相关的事件发生了，它提交它相关的任务代码到分发队列。某些事件，例如 timer source 定期发生，但大部分只有当指定条件出现零星发生。因为这个原因，dispatch source 保留它们相关的分发队列防止它们过早释放。

从上面的介绍我们可以得出使用 dispatch source 时主要就是做三件事：1.指定想要监视的事件；2.提供分发队列；3.编写处理事件的代码。

我们可以使用 dispatch_source_create 函数来指定我们想要监视的事件，如何提供分发队列可以使用分发队列里面介绍的技术，处理事件的代码则可以是 block 对象或者函数。除了这些内容，dispatch source  还贴心的提供了取消功能以及暂停和恢复，所以使用时还得掌握如何取消、暂停和恢复，让我们看个示例总结感受下：

```
// 1. 指定想要监视的事件
dispatch_source_t source = dispatch_source_create(DISPATCH_SOURCE_TYPE_READ,

                                 myDescriptor, 0, myQueue);

// 2. 提供分发队列
dispatch_queue_t myQueue = dispatch_queue_create("com.example.MyCustomSerialQueue", DISPATCH_QUEUE_SERIAL);

// 3. 编写处理事件的代码
dispatch_source_set_event_handler(source, ^{

   // Get some data from the source variable, which is captured
   // from the parent context.
   size_t estimated = dispatch_source_get_data(source);

   // Continue reading the descriptor...

});

dispatch_resume(source);

// 4. 取消
void RemoveDispatchSource(dispatch_source_t mySource)

{
   dispatch_source_cancel(mySource);

   dispatch_release(mySource);
}

// 5. 暂停和恢复
dispatch_suspend(mySource);

dispatch_resume(source);
```

##Reference
o Concurrency Programming Guide    
o [Grand Central Dispatch In-Depth: Part 1/2](http://www.raywenderlich.com/60749/grand-central-dispatch-in-depth-part-1)   
o [Grand Central Dispatch In-Depth: Part 2/2](http://www.raywenderlich.com/63338/grand-central-dispatch-in-depth-part-2)  
o [iOS多线程编程Part 3/3 - GCD](http://www.hrchen.com/2013/07/multi-threading-programming-of-ios-part-3/)

