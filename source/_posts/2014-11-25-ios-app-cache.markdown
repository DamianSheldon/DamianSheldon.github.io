---
layout: post
title: "iOS App Cache"
date: 2014-11-25 16:36:04 +0800
comments: true
published: false
categories: [Archives, iOS Development]
keywords: Cache, iOS App, Data Model, URL Cache, Core Data, SQLite
discription: Summary of iOS App Cache
---
在互联网时代的今天，iOS App几乎都要联网，缓存可以解决网络连接不良甚至无网络所造成的性能问题，而且还可以节约用户的流量。我们可以采用两种缓存策略：预缓存和按需缓存。预缓存就是应用启动以后开个后台线程去把需要用到的数据先取下来。按需缓存就是应用请求网络数据后在本地保存一份，只要本地数据没有过期就使用本地数据。

###预缓存
实现预缓存可能需要一个后台线程访问数据并以有意义的格式保存，以便本地缓存无需连接服务器即可被编辑。Core Data(或者任何结构化存储)是实现这种缓存的一种方式。

###按需缓存
按需缓存工作原理类似于浏览器缓存，它允许我们查看以前访问过的内容，主要有四种实现方法：

1. URL缓存；
2. 数据模型缓存；
3. Core Data;
4. SQLite。

上述的序号是推荐使用的顺序。

####URL缓存
如果服务器设计得体，遵循HTTP 1.1的缓存规范时，URL缓存效果最好，通常网络库会提供支持。

<!-- more -->

####数据模型缓存
数据模型缓存是把模型对象用NSKeyedArchiver归档，模型类需要实现NSCoding协议。

使用数据模型缓存时，有个小技巧，可以为它创建内存缓存。这样有两点好处：一是可以延长闪存的使用寿命；二是可以略微提高性能。


####Core Data
使用Core Data进行按需缓存，需要权衡Core Data的复杂度是否值得。

####SQLite
使用SQLite时，要注意当前的二进制包是否是线程安全的。


##Reference

iOS 6 Programming Push the Limits

