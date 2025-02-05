---
layout: post
title: "MySQL Interview Questions"
date: 2023-01-29 13:57:18 +0800
comments: true
categories: [Archives, Web Development]
keywords: MySQL
description: MySQL Interview Questions
published: false
---

###1.MySQL 索引的底层数据结构？

>Most MySQL indexes (PRIMARY KEY, UNIQUE, INDEX, and FULLTEXT) are stored in B-trees. 

###2.创建索引的注意事项？

* 选择合适的字段创建索引
* 被频繁更新的字段应该慎重建立索引
* 尽可能的考虑建立联合索引而不是单列索引
* 注意避免冗余索引
* 考虑在字符串类型的字段上使用前缀索引代替普通索引
* 避免索引失效
* 删除长期未使用的索引

###3.如何避免复合索引失效？

* 查询条件要遵守最左前缀原则
* 不要在索引列上进行计算、函数、类型转换等操作
* 不要使用 SELECT * 进行查询
* 不要以 % 开头的 LIKE 查询比如 like '%abc'
* 避免查询条件中使用 or，且 or 的前后条件中有一个列没有索引，涉及的索引都不会被使用到
* 避免发生隐式转换

###4.MySQL 默认的隔离级别？

>The default isolation level is REPEATABLE READ. 

幻读问题（Phantom Read），它是指在事务执行过程中，两个完全相同的范围查询得到了不同的结果集。  

不可重复读问题（Non-Repeatable Read），它是指在事务执行过程中，对同一行数据的两次查询得到了不同的结果。  

脏读问题（Dirty Read），它是指在事务执行过程中，一个事务读取到了另一个事务未提交的数据。


