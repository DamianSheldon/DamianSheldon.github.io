---
layout: post
title: "HotSpot 虚拟机垃圾收集调优总结"
date: 2023-02-03 09:34:30 +0800
comments: true
categories: [Archives, Web Development]
keywords: jvm, gc, hotspot
description: HotSpot Virtual Machine Garbage Collection Tuning Summary
---

工作中我们可能需要对 HotSpot 虚拟机垃圾收集调优，那么应该怎么做呢？  

Oracle 在 HotSpot Virtual Machine Garbage Collection Tuning Guide 提到过一个调优策略：  

> Tuning Strategy
> 
> Do not choose a maximum value for the heap unless you know that you need a heap greater than the default maximum heap size. Choose a throughput goal that is sufficient for your application.
> 
> The heap will grow or shrink to a size that will support the chosen throughput goal. A change in the application's behavior can cause the heap to grow or shrink. For example, if the application starts allocating at a higher rate, the heap will grow to maintain the same throughput.
> 
> If the heap grows to its maximum size and the throughput goal is not being met, the maximum heap size is too small for the throughput goal. Set the maximum heap size to a value that is close to the total physical memory on the platform but which does not cause swapping of the application. Execute the application again. If the throughput goal is still not met, then the goal for the application time is too high for the available memory on the platform.
> 
> If the throughput goal can be met, but there are pauses that are too long, then select a maximum pause time goal. Choosing a maximum pause time goal may mean that your throughput goal will not be met, so choose values that are an acceptable compromise for the application.
> 
> It is typical that the size of the heap will oscillate as the garbage collector tries to satisfy competing goals. This is true even if the application has reached a steady state. The pressure to achieve a throughput goal (which may require a larger heap) competes with the goals for a maximum pause time and a minimum footprint (which both may require a small heap).

这个调优的策略比较粗，可以作为我们调优的总纲领，还需要更细化一下才更具可操作性。  

> Selecting a Collector

> Unless your application has rather strict pause-time requirements, first run your application and allow the VM to select a collector.

> If necessary, adjust the heap size to improve performance. If the performance still doesn't meet your goals, then use the following guidelines as a starting point for selecting a collector:

>  * If the application has a small data set (up to approximately 100 MB), then select the serial collector with the option `-XX:+UseSerialGC`. 
> 	* If the application will be run on a single processor and there are no pause-time requirements, then select the serial collector with the option `-XX:+UseSerialGC`. 
> 	* If (a) peak application performance is the first priority and (b) there are no pause-time requirements or pauses of one second or longer are acceptable, then let the VM select the collector or select the parallel collector with `-XX:+UseParallelGC`. 
> 	* If response time is more important than overall throughput and garbage collection pauses must be kept shorter, then select the mostly concurrent collector with `-XX:+UseG1GC`. 
> 	* If response time is a high priority, then select a fully concurrent collector with `-XX:+UseZGC`.
 这份指南就更具操作性了。

周志明老师在他的《深入理解 java 虚拟机》中介绍了如何权衡收集器：  

> 现在可能有读者要犯选择困难症了，我们应该如何选择一款适合自己应用的收集器呢？这个问题主要受以下三个因素影响：  

> * 应用程序的主要关注点是什么?如果是数据分析、科学计算类的任务，目标是尽可能快算出结果，那吞吐量就是主要关注点；如果是 SLA 应用，那停顿时间直接影响服务质量，严重的甚至会导致事务超时，这样延时就是主要关注点；而如果是客户端应用或者嵌入式应用，那垃圾收集的内存占用则是不可忽视的。    

> * 运行应用的基础设施如何？譬如硬件规格，要涉及的系统架构是x86-32/64、SPAR 还是 ARM/Aarch64；处理器的数量多少，分配的内存大小；选择的操作系统是Linux、Solaris 还是 Windows 等。  

> * 使用 JDK 的发行商是什么？版本号是多少？是 ZingJDK/Zulu、OracleJDK、OpenJDK、OpenJ9 抑或是其他公司的发行版? 该 JDK 对应了《 Java 虚拟机规范 》的哪个版本？  

> 一般来说，收集器的选择就从以上这几点出发来考虑。举个例子，假设某个直接面向用户提供服务的 B/S 系统准备选择垃圾收集器，一般来说延迟时间是这类应用的主要关注点，那么：  

> * 如果你有充足的预算但没有太多调优经验，那么一套带有商业技术支持的专有硬件或软件解决方案是不错的选择。Azul 公司以前主推的 Vega 系统和现在主推的 Zing VM 是这方面的代表，这样你就可以使用传说中的 C4 收集器。  

> * 如果你虽然没有充足的预算去使用商业解决方案，但能掌握软硬件型号，使用较新的版本，同时又特别注重延时，那 ZGC 很值得尝试。  

> * 如果你对还处于实验状态的收集器的稳定性有顾虑，或者必须运行在 Windows 操作系统下，那 ZGC 无缘了，试试 Shenandoah 吧。 

> * 如果你接手的是遗留系统，软硬件基础设施和 JDK 版本都比较落后，那就根据内存规模衡量一下，对于大概 4GB 到 6GB 以下的堆内存，CMS 一般能处理得比较好，而对于更大的堆内存，可重点考察一下 G1。  

> 当然，以上都是仅从理论出发的分析，实战中切不可纸上谈兵，根据系统实际情况去测试才是选择收集器的最终依据。 

这里的建议更具体，结合这两份建议，谈谈我个人的理解。  

目前 HotSpot 虚拟机主要有 Serial, Parallel, CMS, Shenandoah 和 ZGC 这几款收集器，Serial, Parallel 主要关注吞吐量; CMS, Shenandoah 和 ZGC 主要关注低延时。  

在做 Java 服务端开发时，我们基本可以不考虑 Serial 收集器。而服务端的应用主要是面向用户提供服务的，所以要选择低延时的收集器。根据使用 java 版本和堆内存大小从 CMS, G1 和 ZGC 中选择，然后进行测试，根据测试结果选择。    

下面举个 Tomcat 的例子，Tomcat 9 支持 java 8 及以上的版本，可以用来试验各自收集器。  

在 Tomcat 的安装包对应的 bin 目录下新建 setenv.sh 文件:  

```
#!/bin/sh
# Set any additional Tomcat options
#CATALINA_OPTS="-Dcatalina.base=$CATALINA_HOME -Dcatalina.home=$CATALINA_HOME"
#CATALINA_OPTS="-XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC -Xloggc:/usr/local/apache-tomcat-9.0.35/logs/parallel-gc.log"
#CATALINA_OPTS="-XX:+UseConcMarkSweepGC -XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC -Xloggc:/usr/local/apache-tomcat-9.0.35/logs/cms-gc.log"
#CATALINA_OPTS="-XX:+UseG1GC -Xlog:gc*=debug:file=/usr/local/apache-tomcat-9.0.35/logs/g1-gc.log"
#CATALINA_OPTS="-XX:+UseZGC -Xlog:gc*=debug:file=/usr/local/apache-tomcat-9.0.35/logs/z-gc.log"
#JDK_JAVA_OPTIONS="--add-opens=java.base/java.lang=ALL-UNNAMED --add-opens=java.base/java.io=ALL-UNNAMED --add-opens=java.rmi/sun.rmi.transport=ALL-UNNAMED"
```

取消对应行的注释去试验对应的收集器，可以考虑使用 ab 之类压测工具观察性能变化，[gceasy](gceasy.io) 可以用来辅助分析 gc 日志。  


##Reference

* [HotSpot Virtual Machine Garbage Collection Tuning Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning)  
* 深入理解 java 虚拟机

