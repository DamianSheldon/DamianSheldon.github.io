---
layout: post
title: "iOS 面试题汇总(一)"
date: 2015-01-05 15:44:07 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS, Technical, Interview, Questions
description: iOS Technical Interview Questions about method swizzling, memory management, thread safe and so on.
---

###1.Explain method swizzling. When you would use it?
A:Method swizzling is a feature of dynamic of Objective-C that use a new method implementation replace the original. When the original implement doesn't meet my need, I will use this technology.

###2.Take three objects: a grandparent, parent and child. The grandparent retains the parent, the parent retains the child and the child retains the parent. The grandparent releases the parent. Explain what happens.
A:It causes strong reference cycle, and results as memory leaks.

###3.What happens when you invoke a method on a nil pointer?
A:返回0

Reference:[nil / Nil / NULL / NSNull](http://nshipster.com/nil/)

###4.Give two separate and independent reasons why retainCount should never be used in shipping code.
A: 

>There should be no reason to explicitly ask an object what its retain count is (see retainCount). The result is often misleading, as you may be unaware of what framework objects have retained an object in which you are interested. In debugging memory management issues, you should be concerned only with ensuring that your code adheres to the ownership rules.

1. 结果不正确，因为你不知道框架中哪些对象已经引用了你感兴趣的对象;
2. 产品代码中不应该包含无用代码。


###5.Explain your process for tracing and fixing a memory leak.
A:

1. Launch Instruments, select Leaks instrument;
2. Use App normal;
3. Notice spike on timeline pane;
4. Check spike is normal;
5. Fix any where that cause memory leak;

We also can use Allocations:

1. Launch Instruments, select Allocations instrument;
2. Use App normal;
3. Make generation when state change;
4. Compare gerations to find out memory leak;
5. Fix memory leak;

Reference:[Instruments Tutorial for iOS: How To Debug Memory Leaks](http://www.raywenderlich.com/2696)



###6.Explain how an autorelease pool works at the runtime level.
A:As its name, it works like a pool, any object called autorelease method be throw to this pool, all object in autorelease pool will recieve a release method when it drain that occured at each event loop end.

###7.When dealing with property declarations, what is the difference between atomic and non-atomic?

A: atomic是指存在竞争赋值时，我们会得到某次完整的赋值，而nonatomic则可能是几次赋值共同组合。

```
@property CGRect domain;

<b>thread 1:</b> puppy.domain = CGRectMake (1.0, 2.0, 3.0, 4.0);
<b>thread 2:</b> puppy.domain = CGRectMake (10.0, 20.0, 30.0, 40.0);

atomic意味着在竞争赋值的情况下得到的结果是CGRectMake (1.0, 2.0, 3.0, 4.0)或者CGRectMake (10.0, 20.0, 30.0, 40.0)。

noatomic情况得到的结果可能是CGRectMake (1.0, 2.0, 30.0, 40.0)这种两次组合的乱码。

```
再进一步，atomic是不是意味着代码是线程安全的呢？不是。atomic修饰符可以保证property的读写操作是串行的，但如果对象的指针不是atomic修饰的，代码仍然不是线程安全的。

###8.In C, how would you reverse a string as quickly as possible?


###9.Which is faster: to iterate through an NSArray or an NSSet?
A: NSArray

Reference:[NSArray or NSSet, NSDictionary or NSMapTable](http://www.cocoawithlove.com/2008/08/nsarray-or-nsset-nsdictionary-or.html)


###10.Explain how code signing works.
A:签名软件先将代码和资源使用单向 hash 算法计算出一系列的 hash 值，之后使用签名者提供的私钥来加密这些 hash 值，加密后的 hash 值存储在代码包中，在代码包中还包含签名者的证书，证书是由 Apple 签发的，能证明签名者的身份，证书里还包含签名者的公钥，操作系统之后可以使用相同的 hash 算法计算代码包里代码和资源的 hash，然后将 hash 值和用公钥解密的 hash 对比，这样就能保证代码并未被修改和确认它的来源，这就是代码签名的大致原理。  


###11.What is posing in Objective-C?

###12.List six instruments that are part of the standard.
A:

1. Time Profiler
2. Leaks
3. Zombies
4. Allocations
5. Activity Monitor
6. Core Animation
7. Network


###13.What are the differences between copy and retain?
A: copy是新创建一个对象副本；retain则是对象引用计数加一。


###14.What is the difference between frames and bounds?

A:

>The frame property contains the frame rectangle, which specifies the size and location of the view in its superview’s coordinate system.
>The bounds property contains the bounds rectangle, which specifies the size of the view (and its content origin) in the view’s own local coordinate system.

###15.What happens when the following code executes? Ball *ball = [[[[Ball alloc] init] autorelease] autorelease];

A: The object gets released twice when the autorelease pool is destroyed.

Reference:[Autoreleasing twice an object](http://stackoverflow.com/questions/11291801/autoreleasing-twice-an-object)

###16.List the five iOS app states.
A:

| State | Description |
| ----- | ----------- |
| Not Running | The app has not been launched or was running but was terminated by the system. |
| Inactive | The app is running in the foreground but is currently not receiving events. (It may be executing other code though.) An app usually stays in this state only briefly as it transitions to a different state.
| Active | The app is running in the foreground and is receiving events. This is the normal mode for foreground apps.
| Background | The app is in the background and executing code. Most apps enter this state briefly on their way to being suspended. However, an app that requests extra execution time may remain in this state for a period of time. In addition, an app being launched directly into the background enters this state instead of the inactive state. For information about how to execute code while in the background
| Suspended | The app is in the background but is not executing code. The system moves apps to this state automatically and does not notify them before doing so. While suspended, an app remains in memory but does not execute any code.

When a low-memory condition occurs, the system may purge suspended apps without notice to make more space for the foreground app.

Reference:  

o [iOS Interview Questions](http://www.raywenderlich.com/53962/ios-interview-questions)

