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
