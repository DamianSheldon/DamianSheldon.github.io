---
layout: post
title: "MySQL 使用笔记(二)"
date: 2020-07-18 15:57:24 +0800
comments: true
categories: [Backend]
keywords: MySQL, procedure, notes 
description: "MySQL 数据库使用笔记（二），继续整理 SQL 优化技巧、索引策略、事务与锁机制、慢查询分析等进阶内容，帮助开发者提升数据库性能调优能力。"
---

###1.如何查看数据库中所有的存储过程?
A: `mysql> show procedure status where db = 'db_for_mysql_crash_course'\G;`;

