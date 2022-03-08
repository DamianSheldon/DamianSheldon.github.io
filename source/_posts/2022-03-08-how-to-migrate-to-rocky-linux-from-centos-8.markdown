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


##Reference:  

* [How to Migrate to Rocky Linux from CentOS Stream, CentOS, Alma Linux, RHEL, or Oracle Linux](https://docs.rockylinux.org/guides/migrate2rocky/)  

