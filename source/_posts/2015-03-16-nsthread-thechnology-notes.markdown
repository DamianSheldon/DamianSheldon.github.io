---
layout: post
title: "iOS Thread笔记"
date: 2015-03-16 16:27:17 +0800
comments: true
categories: [Archives, iOS Development]
keywords: NSThread, NSRunLoop, Concurrent programming
discription: NSThread technology notes
---

1. Thread是什么？
2. iOS中怎么使用Thread？

###Thread是什么

>Threads are a relatively lightweight way to implement multiple paths of execution inside of an application. At the system level, programs run side by side, with the system doling out execution time to each program based on its needs and the needs of other programs. Inside each program, however, exists one or more threads of execution, which can be used to perform different tasks simultaneously or in a nearly simultaneous manner. The system itself actually manages these threads of execution, scheduling them to run on the available cores and preemptively interrupting them as needed to allow other threads to run.

>From a technical standpoint, a thread is a combination of the kernel-level and application-level data structures needed to manage the execution of code. The kernel-level structures coordinate the dispatching of events to the thread and the preemptive scheduling of the thread on one of the available cores. The application-level structures include the call stack for storing function calls and the structures the application needs to manage and manipulate the thread’s attributes and state.

###iOS中怎么使用Thread

iOS中的Thread技术主要有以下两种：

| Technology | Description |
| ---------- | ----------- |
| Cocoa threads | Cocoa implements threads using the NSThread class. Cocoa also provides methods on NSObject for spawning new threads and executing code on already-running threads. 
| POSIX threads | POSIX threads provide a C-based interface for creating threads. If you are not writing a Cocoa application, this is the best choice for creating threads. The POSIX interface is relatively simple to use and offers ample flexibility for configuring your threads.

####Creating a Thread

1. Using NSThread
	* Use the detachNewThreadSelector:toTarget:withObject: class method to spawn the new thread.
	* Create a new NSThread object and call its start method. (Supported only in iOS and OS X v10.5 and later.)

2. Using POSIX Threads
	* OS X and iOS provide C-based support for creating threads using the POSIX thread API. This technology can actually be used in any type of application (including Cocoa and Cocoa Touch applications) and might be more convenient if you are writing your software for multiple platforms. The POSIX routine you use to create threads is called, appropriately enough, pthread_create.
	
3. Using NSObject to Spawn a Thread
	* In iOS and OS X v10.5 and later, all objects have the ability to spawn a new thread and use it to execute one of their methods. The performSelectorInBackground:withObject: method creates a new detached thread and uses the specified method as the entry point for the new thread. 


####Configuring Thread Attributes
Thread的配置项有： 
 
* Stack Size of a Thread  
* Thread-Local Storage
* Detached State of a Thread
* Thread Priority


####Writing Your Thread Entry Routine

编写Thread的入口程序通常需要做如下事项：

* Creating an Autorelease Pool
* Setting Up an Exception Handler
* Setting Up a Run Loop


####Terminating a Thread
