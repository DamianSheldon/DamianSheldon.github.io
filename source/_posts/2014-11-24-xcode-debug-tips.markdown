---
layout: post
title: "Xcode 调试技巧（持续更新）"
date: 2014-11-24 17:14:52 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Debug, iOS App, Xcode
description: Summary of Xcode debug tips
---
iOS App开发过程中不可避免地遇到程序崩溃的问题。当程序崩溃时，我们首先要找到它崩溃的原因。一旦找到原因，问题就容易解决了。Xcode Debugger是查找崩溃原因的有利工具，我们应该学会熟练使用它，迅速解决问题，节约宝贵的开发时间。

###崩溃在main( )

添加Exception Breakpoint

Project > Breakpoint navigator > +（Bottom left）> Add Exception Breakpoint

###符号断点

符号断点是我们验证某个方法是否被调用的一种方法。添加方法：

Project > Breakpoint navigator > +（Bottom left）> Add Symbolic Breakpoint

例如：`application:DidFinishLaunchingWithOptions:`。

###打印方法名

```
NSLog(@"%s", __PRETTY_FUNCTION__);

``` 

###控制台打印

```
(lldb) p // 打印标量变量
(lldb) p (int)self.myAge
(lldb) p (CGPoint)self.view.center


(lldb) po // 打印对象

```
<!-- more -->

###打印异常信息

```
/*
The symbol $eax refers to one of the CPU registers.    
In the case of an exception,   
this register will contain a pointer to the NSException object.   
Note: $eax only works for the simulator,   
if you’re debugging on the device you’ll need to use register $r0.  
*/
// Simulator

(lldb) po [$eax class]

(lldb) po [$eax name]

(lldb) po [$eax reason]

// Real Device
(lldb) po $r0
```

###输出View结构

任意时刻暂停App，在lldb中输入:

```
(lldb) po [[[[UIApplication sharedApplication] delegate] window] recursiveDescription]

```

###SIGABRT

SIGABRT:SIGNAL ABORT(中止信号)。通常可以让程序继续运行，之后会输出些有助于定位问题的信息。

###EXC_BAD_ACCESS

它出现的原因是因为访问一个已经释放的对象或向它发送消息。通常可以开启Zombie Objects(Toolbar > Edit Scheme... > Run > Diagnostics > Enabled Zombie Objects)重新运行程序以定位问题。

Note that you shouldn’t leave Zombie Objects enabled all the time. Because this tool never deallocates memory, but simply marks it as being undead, you end up leaking all over the place and will run out of free memory at some point. So only enable Zombie Objects to diagnose a memory-related error, and then disable it again.

Enabled Zombie Objects后，控制台通常会打印出`*** -[CFNumber respondsToSelector:]: message sent to deallocated instance 0x31ab5cfe0`类似的信息，那么问题来了，我们怎么知道0x31ab5cfe0是哪个对象？

Apple Memory Usage Performace Guidelines中介绍了记录内存分配历史的方法，简述如下：

1. 设置环境变量： MallocStackLogging，MallocStackLoggingNoCompact为1；

{% img /images/Environment.png Environment %}

{% img /images/Zombie.png Zombie %}


2. 使用malloc_history命令找到相应的对象。

```
malloc_history <pid/partial-process-name> [options] <mode> [<address> ...]

// pid/partial-process-name是当前上下文NSLog输出时的前面[]的对应数字
2014-12-02 14:44:39.355 srsApp[7946:300216] selector:0x1014d70b3, jsonValue:0x31a896fd0

malloc_history 5968/224511 0x2d9e23fe0 | grep "0x2d9e23fe0"。
```

{% img /images/Malloc_history.png Malloc_history %}

###unrecognized selector sent to instance 0x7fa71400fc10
A: 

1. Add a exception breakpoint;
2. Check the description of the object in memory address`po (NSObject*)(0x7fa71400fc10)`.

[How to debug “unrecognized selector sent to instance”](http://stackoverflow.com/questions/37928924/how-to-debug-unrecognized-selector-sent-to-instance)

Reference:

Memory Usage Performace Guidelines   
iOS 6 Programming Pushing the Limits  
[My App Crashed, Now What? – Part 1](http://www.raywenderlich.com/10209/my-app-crashed-now-what-part-1)    
[My App Crashed, Now What? – Part 2](http://www.raywenderlich.com/10505/my-app-crashed-now-what-part-2)  
[Intermediate Debugging with Xcode 4.5](http://www.raywenderlich.com/28289/debugging-ios-apps-in-xcode-4-5)     
[Xcode调试技巧](http://www.iwangke.me/2013/01/15/xcode-debugging-tips/)


