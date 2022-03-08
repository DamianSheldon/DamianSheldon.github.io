---
layout: post
title: "CentOS 8 迁移到 Rocky Linux 时遇到的问题"
date: 2022-03-08 11:25:32 +0800
comments: true
categories: 
keywords: Migrate, Rocky Linux, CentOS 8
description: How to Migrate to Rocky Linux from CentOS 8
---

虽然 Rocky Linux 专门写了一篇文档介绍如何从 CentOS 8 迁移过来，但实际迁移还是遇到了问题，这里简单记一下。  

```
Error: 
 Problem 1: cannot install both ibus-libs-1.5.19-14.el8_5.x86_64 and ibus-libs-1.5.19-12.el8.x86_64
  - package ibus-devel-1.5.19-12.el8.x86_64 requires ibus-libs(x86-64) = 1.5.19-12.el8, but none of the providers can be installed
  - cannot install the best update candidate for package ibus-libs-1.5.19-12.el8.x86_64
  - problem with installed package ibus-devel-1.5.19-12.el8.x86_64
 Problem 2: cannot install both marisa-0.2.4-36.el8.x86_64 and marisa-0.2.4-4.el7.x86_64
  - package marisa-devel-0.2.4-4.el7.x86_64 requires marisa(x86-64) = 0.2.4-4.el7, but none of the providers can be installed
  - cannot install the best update candidate for package marisa-0.2.4-4.el7.x86_64
  - problem with installed package marisa-devel-0.2.4-4.el7.x86_64
(try to add '--allowerasing' to command line to replace conflicting packages or '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)
```

第一个问题是升级 `ibus-devel` 失败，这是因为 `ibus-devel` 在 PowerTools 的仓库中，默认可能没有使能这个仓库，我们可以手动使能安装或升级：`sudo dnf --enablerepo=powertools install ibus-devel`。  

第二个问题是 CentOS 8 上有最新的 `marisa-0.2.4-36.el8.x86_64`，但是只有`marisa-devel-0.2.4-4.el7.x86_64`, 这个有点奇怪，不知道为什么两个版本没有同步升级，`marisa-devel` 一般是用于开发，我们可能暂时用不到，可以先尝试卸载它完成升级：`sudo dnf remove marisa-devel`。  

```
Syncing packages

Last metadata expiration check: 0:00:13 ago on Tue Mar  8 12:23:04 2022.

Error: 
 Problem: package kyotocabinet-1.2.77-1.el7.x86_64 requires kyotocabinet-libs(x86-64) = 1.2.77-1.el7, but none of the providers can be installed
  - kyotocabinet-libs-1.2.77-1.el7.x86_64 does not belong to a distupgrade repository
  - problem with installed package kyotocabinet-1.2.77-1.el7.x86_64
(try to add '--skip-broken' to skip uninstallable packages or '--nobest' to use not only best candidate packages)

Error during distro-sync.

An error occurred while we were attempting to convert your system to Rocky Linux. Your system may be unstable. Script will now exit to prevent possible damage.
```

第三个问题是 kyotocabinet 的版本问题导致 `distro-sync` 失败。我查询了一下系统中安装的 kyotocabinet 版本又确实是 `1.2.77-1.el7`，也可以查询到迁移成功了：  

```
$ hostnamectl 
   Static hostname: centos.tenneshop.com
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 2af6da6bd3624e0988bd30e22574b645
           Boot ID: 6edac64aba294e1e9f3b7cfdb7b6970c
    Virtualization: oracle
  Operating System: Rocky Linux 8.5 (Green Obsidian)
       CPE OS Name: cpe:/o:rocky:rocky:8:GA
            Kernel: Linux 4.18.0-240.15.1.el8_3.x86_64
      Architecture: x86-64
```

通过 `Syncing packages` 搜索 migrate2rocky 脚本，这已经是最后三步了，于是尝试手动完成剩下的工作。  先 `dnf --allowerasing distro-sync`，然后查看会删除哪些软件，我这里是会删除 kyotocabinet，根据情况确认是否接受删除软件同步。  

最后的 Disable Stream repos 和移除 `subscription-manage`，因为是 CentOS 8, 所以并不需要。  


##Reference:  

* [How to Migrate to Rocky Linux from CentOS Stream, CentOS, Alma Linux, RHEL, or Oracle Linux](https://docs.rockylinux.org/guides/migrate2rocky/)  

