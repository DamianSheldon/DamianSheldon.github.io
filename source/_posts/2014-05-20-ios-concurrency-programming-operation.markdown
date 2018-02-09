---
layout: post
title: "iOS 并发编程之 Operation"
date: 2014-05-20 09:31:07 +0800
comments: true
categories: [Archives, iOS Development] 
keywords: Operation, NSOperation, Concurrency Programming 
description: iOS 并发编程之 Operation 
---

##背景简介

在计算的早期，计算机单位时间能够执行的最大工作是由 CPU 的时钟频率决定的。但是随着技术的发展和处理器设计得更加小巧，热量和其他物理约束开始限制处理器的最大时钟频率。所以芯片制造商寻找其他办法来提高他们芯片的整体性能。通过增加核数量，单个芯片每秒可以运行更多的指令而不需要增加 CPU 的速度或者改变芯片的尺寸或者热特性。唯一的问题是如何利用额外的核。

应用使用多核的传统方法是创建多个线程。然而，随着核的数量增加，线程方案有很多问题。最大的问题是线程代码不能很好地扩展到任意数量的核。你不能有多少核就创建多少线程然后期望程序能正常运行。你需要知道的是有多少核可以被有效地使用，应用程序依靠自身来计算这是件很有挑战的事情。即使你得到了正确的数量，让如此多的线程高效地运行，相互之间互不影响仍然是一件十分困难的事情。

为了解决这些问题，OS X 和 iOS 采用了 异步设计的方案。其中一种异步开始任务的技术叫做 GCD(Grand Central Dispatch)。该技术接管了你以前需要在应用中编写的线程管理代码，并将它们下移到系统级。所有你需要做的是定义你想做的任务并把它们添加到合适的分发队列。 GCD 负责创建需要的线程并调度你的任务运行到这些线程上。因为现在线程管理是系统的一部分， GCD 为任务管理和运行提供了一个比传统线程更高效的整体方案。

从背景介绍我们可以看出，并发编程的根本还是线程，但是在实际的使用过程中发现在应用层来做线程的管理很困难，编写多线程代码也很难，所以 Apple 从系统级层面对线程做了封装，因此在 OS X 和 iOS 平台我们有了特定的并发编程技术 GCD。

到这里事情并没结束，GCD 是基于 C 接口，Apple 对它用 Objective-C 进行了进一步的包裹和封装，于是有操作队列(Operation Queue)。

操作队列是非常类似分发队列(dispatch queue)的 Objective-C 对象。你定义想要执行的任务并添加到操作队列，它处理这些任务的调度和执行。类似 GCD，操作队列为你处理所有的线程管理，确保任务在系统上高效快速的运行。

因此 iOS 开发中并发编程技术有三种：

1. NSOperation;  
2. GCD;  
3. Thread.  

上述的顺序也是推荐使用的顺序。

##如何使用 Operation?

Operation 的核心思想是把应用想要完成的工作封装起来，然后添加到队列中执行或手动执行。苹果在 Foundation 框架中提供了 NSOperation 这个抽象类，它为我们搭好了用户代码与系统代码交互的骨架，最小化了我们需要做的工作，只需要专注于封装我们想要做的工作。苹果很贴心，为了进一步减轻开发者负担，她还提供了两个具体的类:`NSInvocationOperation` 和 `NSBlockOperation` 用来完成日常大部分工作。  

因此，我们使用 Operation 的方法是用具体的类封装工作或者自定义 Operation 封装工作，具体类可以满足需求时就不用去自定义 Operation 了，这样可以减少我们的工作量，封装好工作之后执行它们，方法有两种：一是加入队列;二是手动执行。  

##使用 NSInvocationOperation 封装工作

``` objective-c
@implementation MyCustomClass
- (NSOperation*)taskWithData:(id)data {
    NSInvocationOperation* theOp = [[NSInvocationOperation alloc] initWithTarget:self
                    selector:@selector(myTaskMethod:) object:data];
 
   return theOp;
}
 
// This is the method that does the actual work of the task.
- (void)myTaskMethod:(id)data {
    // Perform the task.
}
@end
```
<!-- more -->

##使用 NSBlockOperation 封装工作

``` objective-c
NSBlockOperation* theOp = [NSBlockOperation blockOperationWithBlock: ^{
      NSLog(@"Beginning operation.\n");
      // Do some work.
   }];
```

##自定义 Operation 封装工作  

自定义 Operation 之前，我们要清楚两个概念，即非并发 Operation 和并发 Operation：  

1. 非并发 Operation 是指 Operation 在调用线程中同步运行的。  
2. 并发 Operation 是指 Operation 与调用线程是异步运行的。  

当我们把 Operation 加入到 Operation Queue ，之后它底层的线程便会在某个时刻调用 Operation 的 start 方法；手动执行时则是调用代码当前上下文的线程调用 Operation 的 start 方法。当我们想让我们的 Operation 与调用线程异步运行时，我们定义并发 Operation, 例如：你想让 Operation 手动运行时也是异步运行的；又或者让 Operation Queue 的线程主要运行调度代码，真正的任务则跑在自己创建的线程上。反之则定义非并发 Operation。  

###定义非并发 Operation
对于非并发 Operation，你所必须要做的是执行主任务和合适地响应取消事件。

1. 执行主任务；
每个 Operation 对象至少应该实现下面所列的方法：  

	* 自定义的初始化方法
	* main

2. 合适地响应取消事件。
在 Operation 对象中为了支持取消，你所必须要做的是从自己的代码中周期性调用对象的 isCancelled 方法，如果它返回 YES,那么立即返回。当你设计自己的 Operation 对象时，你应该考虑在你代码的下列位置调用 isCancelled 方法。  

	* 在执行真正的任务之前立即调用；
	* 在循环的每一次迭代中至少调用一次，或者更频繁，如果每次迭代相对比较长；
	* 在你代码中相对比较容易中止Operation的任何点。


###定义并发 Operation
对于并发 Operation，你必须用你的自定义代码替换现有的结构。
并发 Operation 需要覆盖如下方法  

| 方法 | 描述  |    
|---- | ----- |  
| start			| (必须的)     
| main			| (可选的） 
| isExecuting 	| (必须的)   
| isFinished 	| (必须的)   
| isConcurrent 	| (必须的)    

##自定义 Operation 的执行行为
Operation 的抽象程度比较高，无论是系统提供的具体类，还是自定义的类，我们都可以配置它执行的一些行为，这是它相较于 GCD 的优势，它可配置的行为有如下这些：  

* 配置内部 Operation 依赖；
* 改变对象的执行优先级；
* 改变底层的线程优先级；
* 设置完成 block.

##执行 Operation 的方法

1. 把 Operations 加入到 Operation Queues;  
2. 手动执行 Operations.

```
// 1. Operation Queues

// Operation Queues 是 NSOperationQueue 的实例，它默认是并发队列，调用setMaxConcurrentOperationCount:，传入参数1可以将队列强制变成串行队列。

// Create Concurrent Operation Queue
NSOperationQueue* aQueue = [[NSOperationQueue alloc] init];

// Force Concurrent Operation Queue To Serail Queue
[aQueue setMaxConcurrentOperationCount:1];

[aQueue addOperation:anOp]; // Add a single operation

[aQueue addOperations:anArrayOfOps waitUntilFinished:NO]; // Add multiple operations

[aQueue addOperationWithBlock:^{

   /* Do something. */

}];

// 2. Executing an operation object manually
[anOp start];

```

##辨析Start和main方法

>start -- Begins the execution of the operation.     
>main -- Performs the receiver’s non-concurrent task.

刚接触 NSOperation 时，对 start 和 main 有点混淆，上面的描述能够清晰地区分它们。无论是手动调用 Operation 还是把它加入到 Operation Queue,首先都会调用 start。当我们的 Operation 是非并发的，我们可以在 main 中定义我们的任务，也就是可以不用覆盖 start 方法，而 Operation 是并发的时候就需要覆盖 start 方法了，但这时是否覆盖 main 方法是可选的。


##Reference
o Concurrency Programming Guide  
o [How To Use NSOperations and NSOperationQueues](http://www.raywenderlich.com/19788/how-to-use-nsoperations-and-nsoperationqueues)     
o [iOS多线程编程Part 2/3 - NSOperation](http://www.hrchen.com/2013/06/multi-threading-programming-of-ios-part-2/)  


