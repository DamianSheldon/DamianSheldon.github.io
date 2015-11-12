---
layout: post
title: "Using MySQL"
date: 2015-08-12 22:01:09 +0800
comments: true
categories: [Archives]
keywords: MySQL
discription: 
---

###1.查看所有的用户

```
select user, host from user;
```
###2.查看用户的权限

```
show grants for 'username'@'localhost';
```

###3.为用户添加操作某个数据库的权限

```
GRANT select, insert, update, delete, index, alter, create ON db2.* TO 'jeffrey'@'localhost';
```


