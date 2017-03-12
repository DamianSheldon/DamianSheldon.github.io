---
layout: post
title: "Android 真机抓包"
date: 2017-03-11 17:04:33 +0800
comments: true
categories: [Archives, Android Development] 
keywords: Root, Tcpdump, ADB, tPacketCapture 
discription: 
---
在 Android 开发过程中，可能会遇到和服务端交互有问题的情况，这时候就得拿出证据来和服务端撕逼, 而最有力的证据自然是抓取的网络数据包；又或者是遇到很诡异的网络问题，这时候就可以借助抓包来分析和定位问题。

如果我们和服务端的交互没有通过 VPN, 而且也不是视频流这种网络性能要求苛刻的情况，我们可以通过 tPacketCapture 这种应用来抓包；

其他情况我们可以通过 root 手机，然后安装 tcpdump 来抓包。

下面我们详细介绍下 通过 tcpdump 抓包这种方法：

* Root 手机

Root 手机的原理是利用系统存在的漏洞来获得 root 权限，[XDA Developers](https://forum.xda-developers.com/) 上有不少 root 工具，很多手机都可以用它们 root。

* 安装 tcpdump

可以到网上搜索为 Android 编译好的 tcpdump 二进制包，例如[这里](http://www.strazzere.com/android/tcpdump)就有一个。

```
// Copy tcpdump to device
$ adb -d push /path/to/tcpdump /sdcard/tcpdump

// Device shell
$ adb -d shell

// Switch to root
$ su

// Copy tcpdump to /data/local/
# cat /sdcard/tcpdump /data/local/tcpdump
```
<!--more-->
* 抓包

```
/# cd /data/local
/# ./tcpdump -i any -p -s 0 -w /sdcard/capture.pcap

//  Options
    # "-i any": listen on any network interface

　　# "-p": disable promiscuous mode (doesn't work anyway)

　　# "-s 0": capture the entire packet

　　# "-w": write packets to a file (rather than printing to stdout)

　　... do whatever you want to capture, then ^C to stop it ...
```

* 分析

```
// Copy capture.pcap to computer
$ adb -d pull /sdcard/capture.pcap /path/to/capture.pcap

Analyze with Wireshark.
```

* Shell Commands

Android 手机上的命令通常不全，我们可以通过安装 BusyBox 来提供一个相对完成 Shell 命令集方便我们的开发工作。

1,Download [BusyBox](http://www.busybox.net/downloads/binaries) 的压缩包;  
2,获取设备 CPU 的架构版本 `adb -d shell cat /proc/cpuinfo `  
3,解开压缩包，把对应 CPU 架构版本的二进制包生命名为 busybox,例如 `mv busybox-armv7l busybox`;  
4,安装 busybox 到设备上，  

```
// Copy busybox to device
$adb -d push /path/to/busybox /sdcard/busybox

// Switch to device shell
$adb -d shell

// Install busybox
$ su

\# cat /sdcard/busybox /system/xbin/busybox

// Check install result
# busybox 

...
```

##Reference

* [Android通过tcpdump抓包](http://www.cnblogs.com/likwo/archive/2012/09/06/2673944.html)  
* [转adb Shell root 权限](http://www.cnblogs.com/blues_/p/3582097.html)  
* [为Android安装BusyBox —— 完整的bash shell](http://www.cnblogs.com/xiaowenji/archive/2011/03/12/1982309.html)  
