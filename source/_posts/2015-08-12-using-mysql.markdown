---
layout: post
title: "Using MySQL"
date: 2015-08-12 22:01:09 +0800
comments: true
categories: 
keywords: 
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

###4.SQL COUNT() Function

SQL COUNT(column_name) Syntax
The COUNT(column_name) function returns the number of values (NULL values will not be counted) of the specified column:

SELECT COUNT(column_name) FROM table_name;
SQL COUNT(*) Syntax
The COUNT(*) function returns the number of records in a table:

SELECT COUNT(*) FROM table_name;
SQL COUNT(DISTINCT column_name) Syntax
The COUNT(DISTINCT column_name) function returns the number of distinct values of the specified column:

SELECT COUNT(DISTINCT column_name) FROM table_name;

###5.SQL Limit

[LIMIT {[offset,] row_count | row_count OFFSET offset}]

```
select Host, Db from db limit 2, 2;

select Host, Db from db limit 2 offset 2;
```
