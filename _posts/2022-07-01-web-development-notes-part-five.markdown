---
layout: post
title: "Web 开发问题汇总(五)"
date: 2022-07-01 09:32:39 +0800
comments: true
categories: [Archives, Web Development]
keywords: jvm, java
description: Noting problems encounter during web development, every fifteen problem produce a blog, this is the fifth.
---

###1.

```
$ jmap -heap 29104
Attaching to process ID 29104, please wait...
ERROR: attach: task_for_pid(29104) failed: '(os/kern) failure' (5)
Error attaching to process: sun.jvm.hotspot.debugger.DebuggerException: Can't attach to the process. Could be caused by an incorrect pid or lack of privileges.
sun.jvm.hotspot.debugger.DebuggerException: sun.jvm.hotspot.debugger.DebuggerException: Can't attach to the process. Could be caused by an incorrect pid or lack of privileges.
	at sun.jvm.hotspot.debugger.bsd.BsdDebuggerLocal$BsdDebuggerLocalWorkerThread.execute(BsdDebuggerLocal.java:169)
	at sun.jvm.hotspot.debugger.bsd.BsdDebuggerLocal.attach(BsdDebuggerLocal.java:287)
	at sun.jvm.hotspot.HotSpotAgent.attachDebugger(HotSpotAgent.java:671)
	at sun.jvm.hotspot.HotSpotAgent.setupDebuggerDarwin(HotSpotAgent.java:659)
	at sun.jvm.hotspot.HotSpotAgent.setupDebugger(HotSpotAgent.java:341)
	at sun.jvm.hotspot.HotSpotAgent.go(HotSpotAgent.java:304)
	at sun.jvm.hotspot.HotSpotAgent.attach(HotSpotAgent.java:140)
	at sun.jvm.hotspot.tools.Tool.start(Tool.java:185)
	at sun.jvm.hotspot.tools.Tool.execute(Tool.java:118)
	at sun.jvm.hotspot.tools.HeapSummary.main(HeapSummary.java:49)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at sun.tools.jmap.JMap.runTool(JMap.java:201)
	at sun.tools.jmap.JMap.main(JMap.java:130)
Caused by: sun.jvm.hotspot.debugger.DebuggerException: Can't attach to the process. Could be caused by an incorrect pid or lack of privileges.
	at sun.jvm.hotspot.debugger.bsd.BsdDebuggerLocal.attach0(Native Method)
	at sun.jvm.hotspot.debugger.bsd.BsdDebuggerLocal.access$100(BsdDebuggerLocal.java:65)
	at sun.jvm.hotspot.debugger.bsd.BsdDebuggerLocal$1AttachTask.doit(BsdDebuggerLocal.java:278)
	at sun.jvm.hotspot.debugger.bsd.BsdDebuggerLocal$BsdDebuggerLocalWorkerThread.run(BsdDebuggerLocal.java:144)
```

A: macOS 上的 java 虚拟机基础工具不能附加到 java 进程上，操作系统环境为:  

```
$ sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.7
BuildVersion:	19H1922
```

java 版本:  

```
$ java -version
java version "1.8.0_121"
Java(TM) SE Runtime Environment (build 1.8.0_121-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.121-b13, mixed mode)
```

google 了一番之后，问题可能是 [macOS 上的 java 8 部分小版本有问题](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=8160376)，于是我打算用虚拟机里面的 RockyLinux 来试下，结果是确定可以。  

<!--more-->

这中间还一个小插曲，也很有意思，这里记一下。虚拟机里面的 RockyLinux 之前并没有安装 java 开发环境，于是我就下载安装 eclipse，2022-06 版本的 eclipse 默认要求 `java 11+` 的运行环境，它默认选择安装的是 java 17，这也确定是一个好选择，一是 ZGC 是从 java 15 开始 Production Ready；二是 java 17 是一个长期支持版本。

>ZGC was initially introduced as an experimental feature in JDK 11, and was declared Production Ready in JDK 15.

RockyLinux 自带的是 java 8，于是我们安装 jdk 8 和 jdk 17，然后使用自带的 alternatives 来切换 java 版本，但是它好像只管理 jre bin 下的命令，并有把 jdk 里面的命令都管理起来。我是使用 jinfo 提示报错发现的:  

```
Type "GenericGrowableArray", referenced in VMStructs::localHotSpotVMStructs in the remote VM, was not present in the remote VMStructs::localHotSpotVMTypes table (should have been caught in the debug build of that VM). Can not continue.
```

可以使用 `alternatives --display java` 去确定一下，从这里也可以看出工具的版本最好对应，所以我们也可以将它们纳入 alternatives 系统来管理。以 jhsdb 为例:  

```
sudo alternatives --add-slave java /usr/lib/jvm/java-17-openjdk-17.0.3.0.7-2.el8_6.x86_64/bin/java /usr/bin/jhsdb jhsdb /usr/lib/jvm/java-17-openjdk-17.0.3.0.7-2.el8_6.x86_64/bin/jhsdb
```

虚拟机里面做开发还是有点卡，最后还是决定在 mac 上安装多个版本的 java，macOS 上并没有自带 centOS 上 alternatives 类似的命令，需要安装第三方软件来管理，之前在 Spring 的源码里看到它是用 SDkMan 来管理多个版本的 java，所以也决定用 SDKMan 来管理。但是 SDKMan 和 alternatives 的实现方式有很大差别，它默认是将软件安装在用户目录下的隐藏目录下，这对 eclipse 之类 IDE 来配置相关的类库会稍微有点不方便，需要开启 Finder 的隐藏文件显示，快捷命令是 `cmd+shift+.`，`defaults write com.apple.Finder AppleShowAllFiles true` 命令暂时没生效。  


Reference:  

* [Mac 使用 jinfo 出现：Can't attach to the process. Could be caused by an incorrect pid or lack of privileg](https://blog.csdn.net/Dongguabai/article/details/88736589)  
* [JDK-8160376 : DebuggerException: Can't attach symbolicator to the process](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=8160376)

###2. [FATAL] [DBT-06103] The port (5,500) is already in use.
在 MacOS 下 Virtualbox 的 RockyLinux 中安装 oracle-xe-21c，详细的错误信息如下：

```
[FATAL] [DBT-06103] The port (5,500) is already in use.
   ACTION: Specify a free port.

Database configuration failed. Check logs under '/opt/oracle/cfgtoollogs/dbca'.
```

参照  

> The short name of your host is missing from /etc/hosts, only the FQDN is there. It should be:  

> ```

> 163.173.24.179  linux.mydomain.com linux

> ```
> linux (hostname -s) is unreachable due to this.

把机器的主机名加入 hosts,   

```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4 centos centos.tenneshop.com
```

重新执行配置命令，问题解决了。

```
Database creation complete. For details check the logfiles at:
 /opt/oracle/cfgtoollogs/dbca/XE.
Database Information:
Global Database Name:XE
System Identifier(SID):XE
Look at the log file "/opt/oracle/cfgtoollogs/dbca/XE/XE.log" for further details.

Connect to Oracle Database using one of the connect strings:
     Pluggable database: centos.tenneshop.com/XEPDB1
     Multitenant container database: centos.tenneshop.com
Use https://localhost:5500/em to access Oracle Enterprise Manager for Oracle Database XE
```

Reference:[[[FATAL]] [[DBT-06103]] The port (5,500) is already in use [duplicate]](https://dba.stackexchange.com/questions/268437/fatal-dbt-06103-the-port-5-500-is-already-in-use)  

