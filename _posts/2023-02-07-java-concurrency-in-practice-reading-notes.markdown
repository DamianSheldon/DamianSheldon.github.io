---
layout: post
title: "Java 并发编程实战读书笔记"
date: 2023-02-07 16:42:01 +0800
comments: true
categories: [Archives, Web Development]
keywords: Concurrency, java, Synchronized, Volatile, Lock, Atomic
description: Java Concurrency in Practice Reading Notes
published: true
---

Java 并发编程实战 (Java Concurrency in Practice) 是一本介绍 Java 并发编程知识的优秀书籍，我阅读几遍后觉得里面的知识如满天繁星，虽然作者在书中穿插了很多总结，但我觉得他没有想完全体系化地来总结 Java 并发编程，而是假设读者有一定的并发编程基础了，本文我试图体系化的来总结一下 Java 并发编程，将这些知识安放到对应的地方，成体系的知识更容易记忆，也更容易应用。  

既然说到并发，那我们首先就要问什么是并发？这需要回顾计算机的发展历史。  

计算机的发明是源于人们对计算的需求，早期的计算就是很狭隘的数学计算，现在已经泛化。

在早期的计算机中不包含操作系统，它们从头到尾只执行一个程序，并且这个程序能访问计算机中的所有资源。在这种裸机环境中，不仅很难编写和运行程序，而且每次只能运行一个程序，这对于昂贵并且稀有的计算机资源来说也是一种浪费。  

那么如何避免这种浪费呢？我们自然而然会想到如果能同时运行多个程序那不就可以避免浪费。计算机先驱自然也想到了这个想法，于是诞生了操作系统。  

操作系统的出现使得计算机每次能运行多个程序，并且不同的程序都在单独的进程中运行：操作系统为各个独立执行的进程分配各种资源，包括内存，文件句柄以及安全证书等。如果需要的话，在不同的进程之间可以通过一些粗粒度的通信机制来交换数据，包括：套接字、信号处理器、共享内存、信号量以及文件等。  

促使进程出现的因素（资源利用率、公平性以及便利性等）同样也促使着线程的出现。线程允许在同一个进程中同时存在多个程序控制流。线程提供了一种直观的分解模式来充分利用多处理器系统中的硬件并行性，而在同一个程序中的多个线程也可以被同时调度到多个CPU上运行。  

并发这个词的字面意思是同时发生，只是这个同时发生是在用户的角度来看，在操作系统这一侧则可能是通过时间片或者调度到不同的处理器核来实现的。这种调度对应就是线程模型，所以并发本质上就是通过使用线程模型来封装我们的计算任务来让它同时发生。  
<!--more-->
并发的主要目的是高效执行任务，而任务执行主要是牵涉到执行机制和任务表达抽象。我们先来看执行机制，它最基础的类是线程，当然可以直接使用线程，但它有一些缺陷，比如线程生命周期的开销非常高，会消耗大量资源。自然会想到重用线程，这样就能平摊开销，标准库提供了 Executor 框架来做这个事。它最基础的接口很简单：

```java
public interface Executor {

    void execute(Runnable command);
}
```

但它有不能解决生命周期问题，于是扩展出了 ExecutorService 接口，它的实现还是各种线程池。

```
Executor (接口)
│
├─ ExecutorService (接口)   —— 可管理生命周期线程池
│   ├─ AbstractExecutorService (抽象类)
│   │    ├─ ThreadPoolExecutor (核心线程池实现)
│   │    │    └─ ScheduledThreadPoolExecutor (定时/周期任务线程池)
│   │    └─ ForkJoinPool (分治任务线程池，直接实现 ExecutorService)
│   └─ Executors (工具类，提供各种线程池工厂方法)
│        ├─ newFixedThreadPool()
│        ├─ newCachedThreadPool()
│        ├─ newSingleThreadExecutor()
│        └─ newScheduledThreadPool()
│
└─ ScheduledExecutorService (接口，继承 ExecutorService)
      └─ ScheduledThreadPoolExecutor (实现类)

```

第 8 章的线程池使用主要就是介绍这块内容。

再来看任务抽象表达。Executor 框架使用 Runnable 作为基本的任务表示形式。Runnable 是一种有很大局限的抽象，虽然 run 能写入到日志文件或者将结果放入某个共享的数据结构，但它不能返回一个值或抛出一个受检查的异常。Callable 是一种更好的抽象：它能返回一个值，并可能抛出一个异常。对于需要获取任务结果或者取消任务则需要 Future, Future表示一个任务的生命周期，并提供了相应的方法来判断是否已经完成或取消，以及获取任务的结果和取消任务等。

有了这些基础设施，现在可以并发执行任务了，但在并发执行任务时，我们可能遇到问题，那么会遇到些什么问题呢？

主要会遇到三类问题：安全性问题，活跃性问题和性能问题。

## 安全性问题

安全性问题是任务内部的正确性问题，和任务执行在哪个线程上没有直接关系。它主要源于任务之间会要共享信息，这时会要解决如何共享信息。解决的思路是优先不共享，其次共享不可变对象，最后是共享可变对象。共享可变对象就要用到同步工具。

不共享意味着独享，像局部变量就只有线程自己能访问，所以它们是线程安全的。  

```

public int loadTheArk(Collection<Animal> candidates) { SortedSet<Animal> animals;
int numPairs = 0;
Animal candidate = null;
// animals confined to method, don’t let them escape!
animals = new TreeSet<Animal>(new SpeciesGenderComparator()); animals.addAll(candidates);
for (Animal a : animals) {
if (candidate == null || !candidate.isPotentialMate(a)) candidate = a;
else {
ark.load(new AnimalPair(candidate, a)); ++numPairs;
candidate = null;
} }
    return numPairs;
}
```
```

另外就是利用 ThreadLocal 为每个线程复制一份变量，这样也能独享。

```
private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>() {
public Connection initialValue() {
return DriverManager.getConnection(DB_URL);
} };
public static Connection getConnection() { return connectionHolder.get();
}
```

但是并不是所有的计算任务都可以不共享可变状态，有的计算任务需要共享状态，这其中有一类是可以共享不可变状态的任务。  

状态可以简单可复杂，简单的就是基本类型，复杂则需要我们自定义类的对象。基本类型加上 final 标识后就是不可变的了，所以这里我们重点需要讨论的是对象。如果某个对象在被创建后其状态就不能被修改，那么这个对象就称为不可变对象。  

当满足以下条件时，对象才是不可变的：  

* 对象创建以后其状态就不能修改。  
* 对象的所有域都是final类型。  
* 对象是正确创建的（在对象的创建期间，this引用没有逸出）。

所以我们定义不可变类时可以遵守上述规则。  

有些任务必须共享可变状态，对这类任务，我们编程时应该做呢？这时候会要求我们类是线程安全的，那如何设计线程安全的类呢？  

我们设计新类时通常是已有的类不能满足需求，这时我们有可能有几种选择，一种是扩展现有的类；另一种则完全定义一个新的类。不管是哪种情况，通常我们是需要为类增加新的字段，让这些字段来表达新的状态，以满足某种新的需求。  

由于设计的类的实例对象会被多个线程共享访问，那么我们必须在多个线程访问之前，之中和之后，对象是处于有效的状态，不然由于对象字段上的数据是错误的，那么程序实现的功能是会有错误。对象在多个线程访问之前要处于有效状态就要求对象是正确的创建。在良好的规范中通常会定义各种不变性条件（Invariant）来约束对象的状态，多个线程访问时则要求对象保持不变性。后验条件（Postcondition）是来描述对象操作的结果，所以多个线程访问之后后验条件应该也没被破坏。  

正确的创建重点是关注在对象创建期间，this 引用没有逸出。而不变性条件和后验条件都需求同步机制，只是可以根据情况选择轻量级的 CAS 或锁，某些情况也可以使用 volatile，这里的重点是站在操作系统这侧去看多个线程同时去访问对象时，它是否可以总是保持它的不变性和后验性。

我们也可以这个角度来审视作者的总结，在设计线程安全类的过程中，需要包含以下三个基本要素：  

* 找出构成对象状态的所有变量。  
* 找出约束状态变量的不变性条件。  
* 建立对象状态的并发访问管理策略。

设计线程安全的类是为了后续来使用，使用则要求能从应用的对象图能访问到它，这称为发布。安全的发布依赖于 Happens-Before 规则。安全发布的常用模式有：  

* 在静态初始化函数中初始化一个对象引用。  
* 将对象的引用保存到volatile类型的域或者AtomicReferance对象中。  
* 将对象的引用保存到某个正确构造对象的final类型域中。  
* 将对象的引用保存到一个由锁保护的域中。

同步工具可以分为两类：一类是阻塞的同步机制，例如锁；另一类是非阻塞的同步机制，例如原子变量；锁属于阻塞的同步机制，拥有锁的线程可以独占地访问变量。锁的缺点是挂起和恢复线程存在很大开销，对于细粒度操作来说不划算。于是产生另一种想法，在硬件层提供原子更新变量的支持，用来支持细粒度操作。

```
阻塞同步工具（Blocking）
├── 1. 锁（Lock）
│   ├── 内置锁（synchronized）
│   └── 显式锁（java.util.concurrent.locks.Lock）
│       ├── ReentrantLock
│       │   ├── 公平锁
│       │   └── 非公平锁
│       ├── ReentrantReadWriteLock
│       │   ├── ReadLock
│       │   └── WriteLock
│       └── StampedLock（悲观锁模式）
│
├── 2. 同步辅助类（Synchronizers）
│   ├── CountDownLatch
│   ├── CyclicBarrier
│   ├── Semaphore
│   ├── Exchanger
│   └── Phaser
│
├── 3. 并发集合（部分阻塞）
│   ├── ConcurrentHashMap（读非阻塞，写部分阻塞）
│   ├── ConcurrentLinkedQueue
│   ├── ConcurrentLinkedDeque
│   ├── CopyOnWriteArrayList（写时阻塞）
│   └── CopyOnWriteArraySet（写时阻塞）
│
└── 4. 其他阻塞工具
    ├── LockSupport（park 阻塞线程）
    └── Thread.join() / wait() / sleep() 等线程阻塞操作

```

```
非阻塞同步工具（Non-blocking / CAS 原子操作）
├── 1. 原子操作类（Atomic）
│   ├── AtomicInteger
│   ├── AtomicLong
│   ├── AtomicReference
│   ├── AtomicIntegerArray / AtomicLongArray
│   ├── AtomicStampedReference（带版本号）
│   └── AtomicMarkableReference（带标记）
│
└── 2. 并发集合（非阻塞实现）
    ├── ConcurrentLinkedQueue（非阻塞链表）
    ├── ConcurrentLinkedDeque
    └── ConcurrentSkipListMap / ConcurrentSkipListSet

```

它们是遵循 JMM 规则实现安全性的具体工具。Java 设计者们设计了 Java 内存模型(JMM)，JMM 规定了 JVM 必须遵循一组最小保证，这组保证规定了对变量的写入操作在何时将对于其他线程可见。

JMM 为程序中所有的操作定义了一个偏序关系，称之为 Happens-Before。要想保证执行操作 B 的线程看到操作 A 的结果（无论 A 和 B 是否在同一个线程中执行），那么在 A 和 B 之间必须满足 Happens-Before 关系。

Happens-Before的规则包括：

* 程序顺序规则。如果程序中操作A在操作B之前，那么在线程中A操作将在B操作之前执行。
* 监视器锁规则。在监视器锁上的解锁操作必须在同一个监视器锁上的加锁操作之前执行。
* volatile变量规则。对volatile变量的写入操作必须在对该变量的读操作之前执行。
* 线程启动规则。在线程上对Thread.Start的调用必须在该线程中执行任何操作之前执行。
* 线程结束规则。线程中的任何操作都必须在其他线程检测到该线程已经结束之前执行，或者从Thread.join中成功返回，或者在调用Thread.isAlive时返回false。
* 中断规则。当一个线程在另一个线程上调用interrupt时，必须在被中断线程检测到interrupt调用之前执行（通过抛出InterruptedException，或者调用isInterrupted和interrupted）。
* 终结器规则。对象的构造函数必须在启动该对象的终结器之前执行完成。
* 传递性。如果操作A在操作B之前执行，并且操作B在操作C之前执行，那么操作A必须在操作C之前执行。


## 活跃性问题

活跃性问题是指：即使程序是安全的，也可能出现不能正常推进的情况。常见的有：

```
活跃性问题
├── 1. 死锁（Deadlock）
│   ├── 多个线程循环等待对方持有的资源，导致任务无法推进
│   ├── 必要条件：互斥、占有且等待、不可抢占、循环等待
│   └── 解决方案：
│       ├─ 避免：资源按顺序申请、使用 tryLock + 超时
│       ├─ 检测：死锁检测工具（如 jconsole, jstack）
│       └─ 解除：破坏其中一条必要条件（如可中断锁）
│
├── 2. 饥饿（Starvation）
│   ├── 线程长期无法获得所需资源（如低优先级线程一直饿死）
│   └── 解决方案：
│       ├─ 公平锁（ReentrantLock(fair=true)）
│       ├─ 合理设置线程优先级
│       └─ 避免资源过度竞争
│
└── 3. 活锁（Livelock）
    ├── 线程虽然在运行，但不断“礼让”，导致任务无法完成
    └── 解决方案：
        ├─ 引入随机等待/退避算法（类似网络重传退避）
        └─ 让协议能最终收敛而非无限尝试

```

## 性能问题

即使程序是安全的、活跃的，如果性能太差，也是不合格的并发程序。常见的性能问题有：

```
性能问题
├── 1. 上下文切换开销
│   ├── 频繁阻塞/唤醒线程 → CPU 花大量时间在线程切换上
│   └── 对策：
│       ├─ 使用线程池，减少线程创建/销毁
│       ├─ 使用非阻塞算法（CAS 原子类）
│       └─ 降低锁粒度，减少锁竞争
│
├── 2. 内存同步开销
│   ├── volatile / synchronized / final 的内存屏障会导致缓存失效
│   └── 对策：
│       ├─ 合理使用 volatile（避免过度使用）
│       ├─ 读多写少场景用 CopyOnWrite 或 ReadWriteLock
│       └─ 使用无锁数据结构（ConcurrentLinkedQueue）
│
├── 3. 锁竞争
│   ├── 多线程争夺同一把锁，导致阻塞等待
│   └── 对策：
│       ├─ 缩小临界区范围（锁分解/锁分段）
│       ├─ 使用读写锁（ReentrantReadWriteLock）
│       ├─ 使用乐观锁（CAS, StampedLock）
│       └─ 减少共享状态，尽量使用不可变对象
│
└── 4. 可伸缩性问题
    ├── 随线程数增加，吞吐量不能线性提升甚至下降
    └── 对策：
        ├─ 使用分段锁（ConcurrentHashMap）
        ├─ 使用无锁/非阻塞算法
        ├─ 避免热点资源（如单点计数器）
        └─ 利用分区/分片思想（LongAdder 替代 AtomicLong）
```
