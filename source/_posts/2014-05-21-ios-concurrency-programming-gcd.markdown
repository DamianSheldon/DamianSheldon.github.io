---
layout: post
title: "iOS Concurrency Programming--GCD"
date: 2014-05-21 11:40:27 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS, Concurrency Programing, GCD
description: Introduce iOS Concurrency Programming GCD
---
##GCD是什么？
GCD是Grand Central Dispatch的缩写，它是用来执行自定义的任务的C接口机制。它可以串行或并行地执行任务，并大大简化了相应的线程代码。它的优点有：  
* 直接简单的编程接口；  
* 自动、整体的线程池管理；  
* 高效的内存管理；  
* 负载时不干扰内核；  
* 异步分发任务到分派队列不会造成死锁；  
* 优雅地处理竞争；  
* 串行分派队列为锁和其他同步操作提供了更高效选择；  

##如何使用GCD？
GCD抽象出来了几组高效的的API,我们使用这些API来完成我们的工作。  
###1）单个任务；
我们通常是通过调用以下API来执行任务：
``` objective-c
dispatch_async
dispatch_async_f
dispatch_sync
dispatch_sync_f
dispatch_after
dispatch_after_f
dispatch_apply
dispatch_apply_f
dispatch_once
```

调用这些API之前，我们还要准备好dispatch queue.通常可以通过以下的API创建和管理Queues:
``` objective-c
dispatch_get_global_queue
dispatch_get_main_queue
dispatch_queue_create
dispatch_get_current_queue
dispatch_queue_get_label
dispatch_set_target_queue
dispatch_main
```

dispatch queues主要有三大类：main queue, Concurrent queue, Serial queue;

i)main queue:通过dispatch_get_main_queue(void)可以取到main queue;

ii)Concurrent queue:通过dispatch_queue_t dispatch_get_global_queue(long priority,unsigned long flags)可以取得全局的并发队列。总共有四个优先级的全局队列：
 DISPATCH_QUEUE_PRIORITY_HIGH        
 DISPATCH_QUEUE_PRIORITY_DEFAULT
 DISPATCH_QUEUE_PRIORITY_LOW        
 DISPATCH_QUEUE_PRIORITY_BACKGROUND

iii）Serial queue:可以使用dispatch_queue_create创建串行或并行队列。

代码示例：
``` objective-c
// i) main queue
dispatch_queue_t mainQueue = dispatch_get_main_queue(void);

// ii)Concurrent Queue
dispatch_queue defaultGlobalConcurrentQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);

// iii)Serial queue
dispatch_queue_t myCustomSerialQueue = dispatch_queue_create("com.example.MyCustomSerialQueue", NULL);

// async
dispatch_async(myCustomSerialQueue, ^{
    printf("Do some work here.\n");
});

// sync
dispatch_sync(myCustomSerialQueue, ^{
    printf("Do some more work here.\n");
});
printf("Both blocks have completed.\n");

// apply
for (i = 0; i < count; i++) {
   printf("%u\n",i);
}

// equlivent implement

dispatch_apply(count, defaultGlobalConcurrentQueue, ^(size_t i) {
   printf("%u\n",i);
});
```

###2）组任务；
可以使用以下API进行组操作：
``` objective-c
dispatch_group_async
dispatch_group_async_f
dispatch_group_create
dispatch_group_enter
dispatch_group_leave
dispatch_group_notify
dispatch_group_notify_f
dispatch_group_wait
```
代码示例：
``` objective-c
// Example 1
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

// Example 2
dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
dispatch_group_t group = dispatch_group_create();
for(id obj in array)
    dispatch_group_async(group, queue, ^{
        [self doWorkOnItem:obj];
    });
dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
dispatch_release(group);
[self doWorkOnArray:array];

```

###3）使用Dispatch Semaphores调控有效的资源使用；
Dispatch Semaphores相关的API如下：
``` objective-c
dispatch_semaphore_create
dispatch_semaphore_signal
dispatch_semaphore_wait
```
代码示例：
```objective-c
// Create the semaphore, specifying the initial pool size
dispatch_semaphore_t fd_sema = dispatch_semaphore_create(getdtablesize() / 2);
 
// Wait for a free file descriptor
dispatch_semaphore_wait(fd_sema, DISPATCH_TIME_FOREVER);
fd = open("/etc/services", O_RDONLY);
 
// Release the file descriptor when done
close(fd);
dispatch_semaphore_signal(fd_sema);
```

##Reference
o Concurrency Programming Guide  
o iOS多线程编程Part 3/3 - GCD;http://www.hrchen.com/2013/07/multi-threading-programming-of-ios-part-3/