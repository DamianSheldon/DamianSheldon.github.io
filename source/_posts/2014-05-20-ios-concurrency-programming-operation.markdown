---
layout: post
title: "iOS 并发编程之 Operation"
date: 2014-05-20 09:31:07 +0800
comments: true
categories: [Archives, iOS Development] 
keywords: Operation, NSOperation, Concurrency Programming 
description: iOS 并发编程之 Operation 
---
iOS 开发中并发的实现方式大致有三种：

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


