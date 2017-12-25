---
layout: post
title: "iOS 开发问题汇总(十)"
date: 2017-09-11 15:42:24 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS 
discription: 
---

###1.在iOS 中如何使用私钥加密数据？
A:当你用私钥加密时它被称为 siging, 之后用公钥解密的过程称为 verify signature.
>Why are you encrypting with the private key? When you encrypt with the private key, that is considered signing not encrypting, becuase it provides no confidentiality. If you want to "encrypt" with the private key, look into data signing, and that should allow you to "encrypt" (read "sign") with the private key and "decrypt" (read "verify signature") with the public key.

Reference:[Encrypting data with a private key on iOS](https://stackoverflow.com/questions/6705928/encrypting-data-with-a-private-key-on-ios)    

###2.  

```
[!] Due to the previous naïve CocoaPods resolver, you were using a pre-release version of `JSONModel`, without explicitly asking for a pre-release version, which now leads to a conflict. Please decide to either use that pre-release version by adding the version requirement to your Podfile (e.g. `pod 'JSONModel', '= 1.2.2P'`) or revert to a stable version by running `pod update JSONModel`.
```
A:今天打开一个旧的的工程，可能由于版本控制的原因，Pods 下的文件都被删除掉了，于是运行 `pod install --verbose`, 安装报了上面的错误。工程是一个公共库，之前都是好的，它依赖了一个私有版本的 JSONModel,这本来也没什么问题，居然报出这么一错误，挺奇怪的。按提示先显示指定版本，依旧报错，看来并不是这个原因；又运行 `pod update JSONModel`,有输出安装私有版本的 JSONModel 的日志，问题解决。不过不清楚为什么会出这种很诡异的问题。

