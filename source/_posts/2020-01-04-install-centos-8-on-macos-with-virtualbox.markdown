---
layout: post
title: "在 macOS 上用 VirtualBox 安装 CentOS"
date: 2020-01-04 13:49:46 +0800
comments: true
categories: [Archives]
keywords: screen flicker, kernel headers not found
description: Record solutions to two problems when install CentOS on macOS with VirtualBox.
---

最近为了更好的实践 Linux，决定在 mac 上使用 VirtualBox 安装一个 CentOS，主要是参考鸟哥的这篇[安裝 CentOS7.x](http://linux.vbird.org/linux_basic/0157installcentos7.php#centos_1)。  

安装之后打开系统出现闪屏，英语应该是称为 screen flicker，google 之后在 VirtualBox 的论坛找到解决方法:

1. 进入单用户维护模式
	
	```
	a. 重启系统
	b. 在菜单选择界面键入 e，进入 grub2 的指令编辑模式
	c. 在指定内核和根文件系统这行最后加上 systemd.unit=rescue.target
	d. 键入 ctrl + x 进入系统
	```
	
2. 强制使用 Xorg
	
	```
	 a. 用 vim 打开 /etc/gdm3/custom.conf
 	 b. 删除 WaylandEnable=false 前的 # 注释符号
 	 c. 保存文件后，systemctl default 来进入正常模式 
	```

解决了闪屏之后，想通过虚拟机菜单中的调整窗口大小来让系统的屏幕全屏发现无用，想起来应该要安装 VirtualBox Guest Additions，于是插入虚拟机提供的光盘来安装。  

首先是提示 kernel headers not found for target kernel 的错误，也提示详细的错误信息位于 /var/log/vboxadd-setup.log，我们可以通过查看该错误日志来找到对应解决方法。于是尝试安装对应的内核头文件，命令为 `yum install kernel-headers kernel-devel`，之后执行 `/sbin/rcvboxadd setup`.

仍然提示 kernel headers not found for target kernel，通过 `uname -r` 和 `rpm -q kernel-headers` 发现版本不一致，于是重启系统选择最新的内核版本。  

再次尝试安装，提示 Error building the module，查看错误日志提示需要安装 `libelf-dev, libelf-devel or elfutils-libelf-devel` ，CentOS 上只有 elfutils-libelf-devel ，安装之后再次安装 VirtualBox Guest Additions。  

提示  

```
ValueError: File context for /opt/VBoxGuestAdditions-6.0.14/other/mount.vboxsf already defined
VirtualBox Guest Additions: Running kernel modules will not be replaced until
the system is restarted
```

这个问题暂时没找到解决方法，但是可以让 CentOS 全屏了，就暂时先不管这个问题了。

Reference:

* [第三章、安裝 CentOS7.x](http://linux.vbird.org/linux_basic/0157installcentos7.php#centos_1)  
* [Ubuntu 1710 screen flicker](https://forums.virtualbox.org/viewtopic.php?f=8&t=85110)  
* [Centos7 Guest Additions fails: kernel headers not found](https://forums.virtualbox.org/viewtopic.php?t=91563)  

