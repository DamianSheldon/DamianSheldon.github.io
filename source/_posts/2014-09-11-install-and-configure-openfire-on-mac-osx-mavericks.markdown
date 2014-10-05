---
layout: post
title: "在Mac OSX Mavericks上安装和配置Openfire"
date: 2014-09-11 09:15:32 +0800
comments: true
categories: [Archives] 
keywords: Openfire, Mac, OSX, Mavericks, Install, Configure
descriptions: Install and configure openfire on Mac OSX mavericks.
---

###安装
  Openfire的官方安装文档并没有详细说明如何在Mac OSX上安装。经过查找，它安装的路径在/usr/local下面，需要更改下它的所有者和用户组，并且openfire.sh也要加上可执行属性。

  常见问题：用浏览器打开localhost:9090，报错。  
  解决办法：查看openfire的日志发现是因为端口被占用了，使用sudo lsof -i:9090命令，查看占用端口的程序，用kill -9 pid，终止它们，通常是安装完openfire后，它默认以root的权限启动了一个副本。再次重启openfire，应该可以正常运行了。

###配置
   配置时需要注意，创建用户时的username，只需填写名字即可，不需要加上@domainname,否则客户端会一直提示密码不正确。查找了很久原因，才知道是这么回事。

