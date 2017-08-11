---
layout: post
title: "iOS 并发编程之 Thread"
date: 2015-03-16 16:27:17 +0800
comments: true
categories: [Archives, iOS Development]
keywords: NSThread, NSRunLoop, Concurrent programming
discription: NSThread technology notes
---

在 iOS 并发编程之 Operation 中，我们说过 GCD 是将线程管理的代码从应用层移到了系统层，它是基于线程技术的。虽然在 iOS 和 OS X 平台不鼓励直接使用线程实现并发，但它还是有它的应用场景。那么什么时候应该使用线程呢？

当我们的代码实时运行时用线程实现是一个好方法。虽然分发队列会尽可能快的执行你的任务，但是它们不能解决实时约束。如果你想让运行在后台的代码给你更可预期的行为，线程是一个更好的选择。

既然线程仍然有它的应用场景，我们还是有必要掌握它。

##Thread是什么

线程是一种在应用中实现多条运行路径相对轻量的方法。在系统层，程序并排运行，系统根据其需要和其他程序的需要为每个程序分配执行时间。但是在每个程序内部存在一个或多个线程运行，它们可以被用来以同时或接近同时的方式执行不同的任务。系统本身实际上管理这些线程的运行，调度它们运行在可用的核上，根据需要中断它们以允许其他线程运行。

从技术角度看, 线程是管理代码执行所需的内核级和应用程序级数据结构的组合。内核级结构将事件分派到线程, 并在一个可用内核上协调线程的抢先调度。应用程序级结构包括用于存储函数调用的调用堆栈和需要管理和操作线程的属性和状态的应用程序结构。

在非并发应用中只有一个运行线程。它在应用的主程序中开始和结束并一个一个地调用不同的方法或函数实现应用的行为。与些相反，支持并发的应用以一个线程开始，根据需要添加创建的附加执行路径。每个新路径有它自定义的开始程序，它和应用的主程序是独立运行的。拥有多线程的应用提供两个重要的潜在优势：

* 多线程可以改善应用的响应性
* 多线程在多核系统上可以改善应用的实时性

如果你的应用只有一个线程，那么它需要做所有的事情。它必须响应事件，更新你应用的窗口和执行实现你应用行为需要的所有计算。单线程的问题在于它一个只有做一件事。当你的某个计算需要花费很长时间完成会发生什么叫？当你的代码忙于计算它需要的值，你的应用停止响应用户的事件和更新窗口。如果这种行为持续足够长的时间，用户也许认为你的应用已经挂起，于是尝试强制退出。但是，如果将自定义计算移到单独的线程上，则应用程序的主线程可以自由地响应用户交互。

在多核计算机如此普遍的今天，线程为某些类型的应用提供了一种提高性能的方法。线程可以在处理器不同的核上同时执行不同的任务，让应用在指定时间增加它工作数量成为了一种可能。

当然，线程并不是解决应用性能问题的万用药。提供好处的同时它也带来各种潜在问题。在应用内拥有多个可执行路径会增加你代码的复杂度。每个线程需要协调它和其他线程的行为以防止损坏应用的状态信息。因为单个应用的线程共享相同的内存空间，它们访问所有相同的数据结构。如果两个线程同时尝试操作相同的数据，一个线程可能覆盖另一个线程的修改，最终导致数据被损坏。即使在这里有了正确防护，你还是要当心编译器优化会在你的代码中引入隐蔽的问题。

##iOS中如何使用Thread

线程编程肯定会牵涉到线程的管理，线程同步，线程间通信，在 iOS 平台，Apple 还带来了它特有的技术 Run Loop。

###线程的管理
线程管理主要是创建线程，设置线程相关的属性，配置线程的事件响应代码和终止线程。

#### 创建线程
iOS中的Thread技术主要有两种：Cocoa 线程 和 POSIX 线程。

* Cocoa 线程
	
	Cocoa 线程是用 NSThread 实现的，还为 NSObject 提供了生成新线程以及在已经运行的线程上执行代码的方法。所以使用 Cocoa 线程有四种:
	
	* 使用 `detachNewThreadSelector:toTarget:withObject:` 类方法生成新线程。
	* 创建一个新 NSThread 对象，调用它的 `start` 方法。（仅支持 iOS 和 OS X v10.5 及以上）
	* 使用 NSObject 的 `performSelectorInBackground:withObject:` 方法生成新线程并用指定的方法作为新线程的入口。（仅支持 iOS 和 OS X v10.5 及以上）
	* 使用 NSObject 的 `performSelector:onThread:withObject:waitUntilDone:` 几个类似方法在指定线程上执行代码。

	
* POSIX 线程
	
	POSIX 线程提供基于 C 接口来创建线程。如果你不是在编写 Cocoa 应用程序，这是你创建线程最好的选择。POSIX 接口使用相对比较简单并且为你配置线程提供了很大的灵活性。
	
	创建 POSIX 线程的方法名为 `pthread_create`，下面是个示例:
	
```
#include <assert.h>

#include <pthread.h>

void* PosixThreadMainRoutine(void* data)
{
    // Do some work here.
    
    return NULL;
}

void LaunchThread()
{
    // Create the thread using POSIX routines.
    pthread_attr_t  attr;

    pthread_t       posixThreadID;

    int             returnVal;

    returnVal = pthread_attr_init(&attr);
    assert(!returnVal);

    returnVal = pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
    assert(!returnVal);

    int     threadError = pthread_create(&posixThreadID, &attr, &PosixThreadMainRoutine, NULL);

    returnVal = pthread_attr_destroy(&attr);
    assert(!returnVal);

    if (threadError != 0) {
         // Report an error.
    }
}
```

####设置线程相关的属性
线程的配置项有： 
 
	* 线程的栈大小
	* Thread-Local 存储
	* 线程的 Detached 状态
	* 线程优先级

设置线程的栈大小

对于每个你创建的新线程，系统在你进程空间分配指定大小的内存充当它的栈。栈管理栈帧也是声明局部变量的地方。如果你想改变指定线程栈的大小，你需要在创建它之前就改变。所有的线程技术都提供一些设置栈大小的方法，虽然使用 NSThread 设置大小只在 iOS 和 OS X v10.5 及以上可用。下表列出了每种技术不同的方法。

| 技术 | 方法 |
| --- | ---- |
| Cocoa | 在 iOS 和 OS X v10.5 及以上，分配和初始化一个 NSThread 对象(不要使用 `detachNewThreadSelector:toTarget:withObject:` 方法)。在调用 `start` 方法之前，使用 `setStackSize:`方法来指定新的栈大小。 |
| POSIX | 创建新的 `pthread_attr_t` 并使用 `pthread_attr_setstacksize` 来改变默认的栈大小。当创建线程时传递该属性给 `pthread_create` 函数。 |

配置 Thread-Local 存储

每个线程维护了一个在线程任务地方都可以存取的键-值对字典。你可以使用这个字典存储你在想在你线程整个运行期间持久化的信息。例如，你可以使用它存储那些你想在你线程运行循环多次遍历中保持的状态信息。

Cocoa 和 POSIX 用不同的方法存储线程字典，所以你不能混用这两种技术。只要你在线程代码中坚持使用相同的技术，最终的结果是相似的。在 Cocoa 中，你使用 NSThread 对象的 `threadDictionary` 方法来获取 NSMutableDictionary 对象，你可以使用它来添加你线程需要键。在 POSIX 中，你使用 `pthread_setspecific`和`pthread_getspecific` 函数来存取你线程的键值。

设置线程的 Detached 状态

大多数上层线程技术默认创建 detached 的线程。在大多数情况下，detached 线程是更好的，因为他们允许系统在线程完成时立即释放线程的数据结构。Detached 线程不需要显示地与你的程序交互。从线程获取结果的方法是留给你自己决定。相反地，系统不会回收 joinable 的线程的资源直到其它线程显示地 join 它, 进程可能会阻塞执行 join 的线程。

你可以把 joinable 线程想像成子线程。虽然他们仍然是独立运行的线程，一个 joinable 线程在系统回收资源前必须被另一个线程 join。在退出前，一个 joinable 线程可以传递一个数据指针或其它返回值给 pthread_exit 函数。另一个线程然后可能得到这个数据通过调用 pthread_join 函数。

如果你确实想创建 joinable 线程，唯一的方法是使用 POSIX 线程。POSIX 默认创建的线程就是 joinable 。为了标记线程是 detached 或者 joinable ，在创建线程前使用 `pthread_attr_setdetachstate` 函数修改线程的属性。

设置线程的优先级

任何你新创建的线程都有一个默认的优先级。内核的调度算法在决定哪个线程运行时会把线程的优先级考虑进来，优先级高的比优先级低的更可能运行。优先级并不会保证执行指定时长，仅仅是和低优先级比起来更可能被调度器选中。

如果你想修改线程的优先级，Cocoa 和 POSIX 都提供了方法。对于 Cocoa 线程，你可以使用 NSThread 的类方法 `setThreadPriority:` 来改变当前运行线程的优先级。对于 POSIX 线程，你使用 `pthread_setschedparam` 函数。

####配置线程的事件响应代码



####终止线程

退出线程推荐的方法是让它正常地退出入口程序。虽然 Cocoa， POSIX 提供程序直接杀掉线程，使用这些程序是强烈不推荐的。杀掉线程会妨碍线程清理它自己。线程分配的内存可能会潜在泄露，并且正常使用的资源可能不会被正确的回收，导致潜在的问题。

如果你需要在中途终止线程，你应该设计你的线程可以响应取消和退出消息。对于长时间运行的操作，这可能是周期性的停止工作然后检查有没有这种消息。如果收到请求退出的消息，线程拥有机会执行任务清理工作并优雅地退出；如果没有，它可以返回继续工作并处理后续的数据块。

一种响应取消消息的方法是使用运行循环输入源来接收这样的消息。

###Run Loop

###线程间通信

###线程同步

## Reference

* Thread Programming Guide

