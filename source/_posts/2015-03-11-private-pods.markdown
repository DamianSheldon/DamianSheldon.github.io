---
layout: post
title: "如何创建私有的Pod"
date: 2015-03-11 17:23:04 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Cocoa pods, private pods
discription: How to create a private pod
---

公司的代码经常是需要保密的，我们可以创建私有的pod来方便我们的日常开发工作。  

Cocoa Pods工作依赖于两个文件：  
1. podspec:一个pod的配置是什么，pod的代码放在哪里
2. Podfile:项目依赖哪个pod，以何种方式依赖，它的podspec放在哪里  

Cocoa Pods提供创建静态库pod的方法pod lib create [Pod name]，但这次我由于各种原因不是这样开始的，而是手动先创建了静态库，然后再新建开发工程的方式。于是我需要手动创建podspec文件:

```
pod spec create --verbose project
```
<!--more-->

然后编辑podspecs文件，这里有几个地方需要注意：

```
// s.source指定工程的源码地址,这里路径有些怪异是因为我的git server是布署在Windows上的。
s.source  = { :git => "ssh://dongmeiliang@192.168.1.100:/git/ICW/Git/AJFrame.iOS", :tag => "2.0.0" }

// s.source_files指定哪些源文件需要包含在pod中
s.source_files  = "AJFrame.iOS/*.{h,m}"

// pod的依赖
s.dependency "JSONKit", "~> 1.4"

```

podspecs完成之后, pod spec lint命令可以检查有没有错误。接下来在需要依赖这个pod的工程的Podfile文件中指定podspec的存放路径。

```
pod 'AJFrame.iOS', :git => 'ssh://192.168.1.100:/git/ICW/Git/AJFrame.iOS', :tag => '2.0.0'
```

###Reference  

[Using Pod Lib Create](http://guides.cocoapods.org/making/using-pod-lib-create)  
[The Podfile](http://guides.cocoapods.org/using/the-podfile.html)  
[CocoaPods进阶：本地包管理](http://www.iwangke.me/2013/04/18/advanced-cocoapods/)  
[用CocoaPods做iOS程序的依赖管理](http://blog.devtang.com/blog/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)  
[Cocoapods 入门](http://studentdeng.github.io/blog/2013/09/13/cocoapods-tutorial/)

