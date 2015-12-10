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
$ mysql -u user -p
mysql > USE mysql;
mysql > SELECT user, host FROM user;
```
###2.查看用户的权限

```
mysql > SHOW GRANTS FOR 'username'@'localhost';
```

###3.为用户添加操作某个数据库的权限

```
GRANT select, insert, update, delete, index, alter, create ON db2.* TO 'jeffrey'@'localhost';
```

###4.Default options are read from the following files in the given order:
/etc/my.cnf /etc/mysql/my.cnf /usr/local/mysql/etc/my.cnf ~/.my.cnf

###5.

```
mysql > CREATE TABLE IF NOT EXISTS categories (
	id SMALLINT NOT NULL AUTO_INCREMENT, 
	category VARCHAR(30) NOT NULL, 
	PRIMARY KEY (id), 
	UNIQUE KEY category (category)
)ENGINE = MyISAM DEFAULT CHARSET = utf8;
```

* VARCHAR(30):can hold up to 30 characters.  
* A UNIQUE index creates a constraint such that all values in the index must be distinct. An error occurs if you try to add a new row with a key value that matches an existing row. For all engines, a UNIQUE index permits multiple NULL values for columns that can contain NULL.  
* DEFAULT CHARSET

<!-- more -->
###6.

```
mysql > CREATE TABLE IF NOT EXISTS orders (
	id INT UNSIGNED NOT NULL, 
	user_id INT UNSIGNED NOT NULL, 
	transaction_id VARCHAR(19) NOT NULL, 
	payment_status VARCHAR(15) NOT NULL, 
	payment_amount DECIMAL(6,2) UNSIGNED NOT NULL, 
	payment_date_time TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP, 
	PRIMARY KEY(id), 
	KEY user_id (user_id)
)ENGINE = MyISAM DEFAULT CHARSET = utf8;
```

* KEY is normally a synonym for INDEX. The key attribute PRIMARY KEY can also be specified as just KEY when given in a column definition. This was implemented for compatibility with other database systems.  
* INDEX

