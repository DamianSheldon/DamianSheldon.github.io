---
layout: post
title: "MySQL 使用笔记(二)"
date: 2020-07-18 15:57:24 +0800
comments: true
categories: [Archives, Web Development]
keywords: MySQL, procedure, notes 
description: MySQL notes
---

###1.如何查看数据库中所有的存储过程?
A: `mysql> show procedure status where db = 'db_for_mysql_crash_course'\G;`;

