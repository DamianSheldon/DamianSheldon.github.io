---
layout: post
title: "如何创建私有的Pod"
date: 2015-03-11 17:23:04 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Cocoa pods, private pods
discription: How to create a private pod
---

公司的框架代码大多数时候是需要保密的，而手动的复制导入比较麻烦，这时候我们可以创建私有 Pod 来方便我们的日常开发工作，本文是我创建私有 Pod 的笔记。

Cocoapods 提供了 pod lib create [pod name] 来创建私有库工程，但是有个 embedded dylibs/frameworks are only supported on iOS 8.0 and later 的问题。 意思是只有部署 iOS 8 以上的应用才能使用它，所以要支持 iOS 8 以下的应用的 Pod 我们还是得手动创建，详细步骤如下：

```
$ mkdir AJFrame

// Create static library
Xcode > File > New > Project... > Framework & Library > Cocoa Touch Static Libray 

// Create dev app
Xcode > File > New > Project... > Single View 

// Result 
AJFrame
├── AJFrame
│   ├── AJFrame.xcodeproj
│   ├── AJFrameTests
│   ├── Classes
├── AJFrame.podspec
├── AJFrameDevApp
│   ├── AJFrameDevApp
│   ├── AJFrameDevApp.xcodeproj
│   ├── AJFrameDevApp.xcworkspace
│   ├── AJFrameDevAppTests
│   ├── Podfile
│   ├── Podfile.lock
│   └── Pods
└── LICENSE
```
<!-- more -->
Cocoapods 工作依赖于两个文件：  

1. podspec:一个pod的配置是什么，pod的代码放在哪里  
2. Podfile:项目依赖哪个pod，以何种方式依赖，它的podspec放在哪里    

###创建podspec文件

```
pod spec create [NAME|https://github.com/USER/REPO] --verbose
```
<!--more-->

然后编辑podspecs文件，具体可以参考它的[详细语法](https://guides.cocoapods.org/syntax/podspec.html)，这里有几个地方需要注意：

```
// s.source指定工程的源码地址,这里路径有些怪异是因为我的git server是布署在Windows上的。
s.source  = { :git => "ssh://dongmeiliang@192.168.1.100:/git/ICW/Git/AJFrame.git", :tag => "2.0.0" }

// s.source_files指定哪些源文件需要包含在pod中
s.source_files  = "AJFrame/classes/*.{h,m}"

// pod的依赖
s.dependency "JSONKit", "~> 1.4"

```
可以使用命令 `pod lib lint --verbose` 来验证 podspecs 文件。

###开发静态库
编辑辅助开发静态库应用工程的 Podfile，具体可以参考它的[详细语法](https://guides.cocoapods.org/syntax/podfile.html) ,让它使用安装链接静态库.

```
target 'AJFrameDevApp' do
    pod 'AJFrame', :path => '../'
end

$ cd ~/Documents/AJFrame/AJFrameDevApp/
$ pod install --no-repo-update --verbose
```

###测试
开发完成以后，可以创建一个测试工程来测试私有 Pod 是否能正常工作，具体而言,通常是在创建一个远程仓库，把私有 Pod 的代码推送上去，然后创建一个新的工程，让它安装这个私有 Pod ，然后写些测试代码，测试它能否正常工作。

```
// Remote 
$ cd /Git
$ git init --bare AJFrame.git

// Local
$ cd ~/Documents/AJFrame
$ git init 
$ git add .
$ git commit
$ git tag <tagname>
$ git remote add origin <url>
$ git push -u origin master --tags --verbose

// Edit dependency project's Podfile, add a similary line
pod 'AJFrame', :git => 'ssh://192.168.1.100:/git/ICW/Git/AJFrame.git', :tag => '2.0.0'
$ pod install --no-repo-update --verbose
$ write code and test
```

###发布
发布是可选的，但是当我们有其他的 Pod 想依赖开发的这个私有 Pod 时就变成必须的，所以我们还是得会发布一个私有 Pod。具体步骤如下：

```
// Create a Private Spec Repo  
$ cd /Git  
$ git init --bare Specs.git

// Add your Private Repo to your CocoaPods installation
$ pod repo add aijian-specs ssh://192.168.1.105:/git/ICW/Git/Specs.git --verbose

// Check your installation is successful and ready to go
$ cd ~/.cocoapods/repos/aijian-specs
$ pod repo lint .

// Add your Podspec to your repo
pod repo push aijian-specs AJFrame.podspec

```

在工程中使用它时，编辑 Podfile， 加入如下内容：

```
source ssh://192.168.1.105:/git/ICW/Git/Specs.git

pod 'AJFrame', '~> 2.0.0' 
```

### s.dependency 如何依赖私有 Pod？

1. Create a Private Spec Repo  

```
$ cd /opt/git  
$ mkdir Specs.git  
$ cd Specs.git
$ git init --bare
```

2. Add your Private Repo to your CocoaPods installation 

```
$ pod repo add artsy-specs git@github:artsy/Specs.git

$ pod repo add aijian-specs ssh://192.168.1.105:/git/ICW/Git/Specs.git --verbose
``` 

Check your installation is successful and ready to go:

```
$ cd ~/.cocoapods/repos/artsy-specs
$ pod repo lint .
```

3. Add your Podspec to your repo

```
pod repo push artsy-specs ~/Desktop/Artsy+OSSUIFonts.podspec
```
Your private Pod is ready to be used in a Podfile. You can use the spec repository with the source directive in your Podfile as shown in the following example:

```
source 'URL_TO_REPOSITORY'
```

[Solution Reference](http://stackoverflow.com/questions/25759170/how-to-add-a-private-cocoapod-as-a-dependency-in-another-pod-podspec-file)

###Reference  

[Using Pod Lib Create](http://guides.cocoapods.org/making/using-pod-lib-create)  
[The Podfile](http://guides.cocoapods.org/using/the-podfile.html)  
[CocoaPods进阶：本地包管理](http://www.iwangke.me/2013/04/18/advanced-cocoapods/)  
[用CocoaPods做iOS程序的依赖管理](http://blog.devtang.com/blog/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)  
[Cocoapods 入门](http://studentdeng.github.io/blog/2013/09/13/cocoapods-tutorial/)

