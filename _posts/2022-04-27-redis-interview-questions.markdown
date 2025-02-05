---
layout: post
title: "Redis 面试题"
date: 2022-04-27 14:36:10 +0800
comments: true
categories: [Archives, Web Development]
keywords: Redis, Cache
description: Redis Interview Questions
---

##1.什么是 Redis?

>Redis is an open source (BSD licensed), in-memory data structure store used as a database, cache, message broker, and streaming engine.

Redis是一个开源的内存数据结构存储，被用作数据库、缓存、消息代理和流媒体引擎。

##2.Redis的内存占用量是多少？
我们看几个例子（都是使用64位实例得到的。

* 一个空的实例使用~3MB的内存。
* 100万个小键->字符串值对使用~85MB的内存。
* 100万个键->哈希值，代表一个有5个字段的对象，使用~160 MB的内存。

##3.为什么Redis将其整个数据集保存在内存中？
在过去，Redis的开发者尝试了虚拟内存和其他系统来允许大于RAM的数据集，但最终我们很高兴能做好一件事：数据从内存中提供，磁盘用于存储。所以现在还没有计划为Redis创建一个磁盘上的后端。Redis是当前设计的结果。

##4.你能将Redis与基于磁盘的数据库一起使用吗？
可以，一个常见的设计模式是在Redis中保存频繁写的小数据（以及你需要Redis数据结构以有效方式为你的问题建模的数据），而大块的数据保存到 SQL或最终一致的磁盘数据库。同样，有时Redis被用来在内存中获取存储在磁盘数据库中的相同数据子集的另一个副本。这看起来类似于缓存，但实际上是一个更高级的模型，因为通常Redis数据集与磁盘数据库数据集一起更新，而不是在缓存错过时刷新。

##5.如何减少Redis的整体内存用量？
如果可以的话，使用Redis的32位实例。同时善用小的哈希值、列表、排序集和整数集，因为Redis能够在少数元素的特殊情况下以更紧凑的方式表示这些数据类型。

##6.如果Redis的内存用完了会怎样？
Redis有内置的保护措施，允许用户设置内存使用的最大限制，使用配置文件中的maxmemory选项，对Redis可以使用的内存进行限制。如果达到这个限制，Redis将开始对写命令回复错误（但会继续接受只读命令）。

你也可以配置Redis在达到最大内存限制时驱逐键值。
<!--more-->
##7.Redis的磁盘快照是原子的吗？
是的，Redis的后台保存过程总是在服务器没有执行命令时创建，所以每个在RAM中报告为原子的命令从磁盘快照的角度看也是原子的。

##8.Redis如何使用多个CPU或核心？
CPU成为你使用Redis的瓶颈并不是很常见，因为通常Redis的问题不是内存就是网络带宽。例如，当使用 piplelining 时，在一个普通的Linux系统上运行的Redis实例每秒可以提供100万个请求，所以如果你的应用程序主要使用O(N)或O(log(N))命令，它几乎不会使用太多的CPU。

然而，为了最大限度地提高CPU的使用率，你可以在同一个机器里启动多个Redis的实例，并把它们当作不同的服务器。在某些时候，一个机器可能无论如何也不够用，所以如果你想使用多个CPU，你可以提前开始考虑一些方法来分片。

##9.一个Redis实例能容纳的最大键数是多少？Hash、List、Set和Sorted Set中元素的最大数量是多少？
Redis最多可以处理`2^32`个键，经实践测试，每个实例至少可以处理2.5亿个键。

每个哈希、列表、集合和排序集，都可以容纳`2^32`个元素。

换句话说，你的极限可能是你系统中的可用内存。

##10.为什么我的复制节点与主实例的键数量不同？

如果你使用有效期有限的键（Redis过期），这是正常行为。下面是问题的原因：

* 主节点在与复制节点的第一次同步中生成一个RDB文件。
* RDB文件将不包括在主服务器中已经过期但仍在内存中的密钥。
* 这些键仍然在Redis主节点的内存中，即使在逻辑上已经过期。它们将被认为是不存在的，它们的内存将在以后被回收，要么是增量的，要么是访问时明确的。虽然这些键在逻辑上不是数据集的一部分，但它们在INFO输出和DBSIZE命令中被计算在内。
* 当复制节点读取由主文件生成的RDB文件时，这组键将不会被加载。

正因为如此，对于有很多过期键的用户来说，在复制节点中看到较少的键是很常见的。然而，从逻辑上讲，主节点和复制节点将有相同的内容。

##11.Redis 哨兵和集群的区别是什么？

>Redis Sentinel provides high availability for Redis when not using Redis Cluster.
>Redis scales horizontally with a deployment topology called Redis Cluster.

##12.Memcache 与 Redis 的区别都有哪些？

* Command-Line
* Disk I/O Dumping
* Data Structures
* Replication
* Transactions
* Publish and Subscribe Messaging
* Geospatial Support
* Architecture
* LUA Scripting
* Memory Usage

* [Memcached vs Redis](https://www.baeldung.com/memcached-vs-redis)  

