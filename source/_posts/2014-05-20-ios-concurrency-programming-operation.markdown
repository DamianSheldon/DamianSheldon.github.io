---
layout: post
title: "iOS Concurrency Programming--Operation"
date: 2014-05-20 09:31:07 +0800
comments: true
categories: [Archives, iOS Development] 
keywords: iOS, Concurrency Programming, Operation
description: Introduce iOS Concurrency Programming Operation
---
iOS Concurrency 的实现方式大致有3种：1,NSOperation;2,GCD;3,Thread.上述的顺序也是推荐使用的顺序。

Operation对象是NSOperation类的实例，用来封装你想让应用做的工作。

##Foundation框架中的Operation类
1. NSInvocationOperation类方便我们基于应用的对象和选择器创建operation对象。  
2. NSBlockOperation类方便我们同时执行一个或多个block对象。  
3. NSOperation类是自定义operation对象的基类。  

##并发 VS 非并发Operation
1. 并发Operation是指Operation在调用线程中同步运行的。  
2. 非并发Operation是指Operation与调用线程是异步运行的。

##创建一个NSInvocationOperation对象
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

##创建一个NSBlockOperation对象

``` objective-c
NSBlockOperation* theOp = [NSBlockOperation blockOperationWithBlock: ^{
      NSLog(@"Beginning operation.\n");
      // Do some work.
   }];
```

##定义一个专属的Operation对象
###非并发Operation对象
对于非并发Operation，你所必须要做的是执行主任务和合适地响应取消事件。

1. 执行主任务；
每个Operation对象至少应该实现下面所列的方法：
	* 自定义的初始化方法
	* main

2. 合适地响应取消事件。
在Operation对象中为了支持取消，你所必须要做的是从自己的代码中周期性调用对象的isCancelled方法，如果它返回YES,那么立即返回。当你设计自己的Operation对象时，你应该考虑在你代码的下列位置调用isCancelled方法。
	* 在执行真正的任务之前立即调用；
	* 在循环的每一次迭代中至少调用一次，或者更频繁，如果每次迭代相对比较长；
	* 在你代码中相对比较容易中止Operation的任何点。


###并发Operation对象
对于并发Operation，你必须用你的自定义代码替换现有的结构。
并发Operation需要覆盖的方法  

| 方法 | 描述  |    
|---- | ----- |  
| start			| (必须的)     
| main			| (可选的） 
| isExecuting 	| (必须的)   
| isFinished 	| (必须的)   
| isConcurrent 	| (必须的)    

##自定义Operation对象的执行行为
* 配置内部Operation依赖；
* 改变对象的执行优先级；
* 改变底层的线程优先级；
* 设置完成block.

##执行Operation的方法

1. 把Operations加入到Operation Queues;  
2. 手动执行Operations.

##Operation Queues

Operation Queues是NSOperationQueue的实例，它默认是并发队列，调用setMaxConcurrentOperationCount:，传入参数1可以将队列强制变成串行队列。

```
// Create Concurrent Operation Queue
NSOperationQueue* aQueue = [[NSOperationQueue alloc] init];

// Force Concurrent Operation Queue To Serail Queue
[aQueue setMaxConcurrentOperationCount:1];
```


##辨析Start和main方法
>start -- Begins the execution of the operation.     
>main -- Performs the receiver’s non-concurrent task.

刚接触NSOperation时，对start和main有点混淆，上面的描述能够清晰地区分它们。无论是手动调用Operation还是把它加入到Operation Queue,首先都会调用start。当我们的Operation是非并发的，我们可以在main中定义我们的任务，也就是可以不用覆盖start方法，而Operation是并发的时候就需要覆盖start方法了，但这时是否覆盖main方法是可选的。


##Reference
o Concurrency Programming Guide  
o [How To Use NSOperations and NSOperationQueues](http://www.raywenderlich.com/19788/how-to-use-nsoperations-and-nsoperationqueues)     
o [iOS多线程编程Part 2/3 - NSOperation](http://www.hrchen.com/2013/06/multi-threading-programming-of-ios-part-2/)  
