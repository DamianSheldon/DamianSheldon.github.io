---
layout: post
title: "Using MySQL"
date: 2015-08-12 22:01:09 +0800
comments: true
categories: [Archives, Web Development]
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

###7.--log-bin option
Q:How to check the --log-bin status of MySQL?
A:

> To make incremental backups, we need to save the incremental changes. In MySQL, these changes are represented in the binary log, so the MySQL server should always be started with the --log-bin option to enable that log.

Q:How to enable the --log-bin option of MySQL?
A:To enable the binary log, start the server with the --log-bin[=base_name] option. If no base_name value is given, the default name is the value of the pid-file option (which by default is the name of host machine) followed by -bin. If the base name is given, the server writes the file in the data directory unless the base name is given with a leading absolute path name to specify a different directory. It is recommended that you specify a base name explicitly rather than using the default of the host name; 

###8.You have enabled the binary log, but you haven't provided the mandatory server-id.

```
2016-01-27T14:15:24.323616Z 0 [Note] /usr/local/mysql/bin/mysqld (mysqld 5.7.9-log) starting as process 9709 ...
2016-01-27T14:15:24.327264Z 0 [Warning] Setting lower_case_table_names=2 because file system for /usr/local/mysql/data/ is case insensitive
2016-01-27T14:15:24.327888Z 0 [ERROR] You have enabled the binary log, but you haven't provided the mandatory server-id. Please refer to the proper server start-up parameters documentation
2016-01-27T14:15:24.328034Z 0 [ERROR] Aborting

2016-01-27T14:15:24.328049Z 0 [Note] Binlog end
2016-01-27T14:15:24.328153Z 0 [Note] /usr/local/mysql/bin/mysqld: Shutdown complete
```

A: set server-id

```
sudo /usr/local/mysql/bin/mysqld --user=_mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --log-error=/usr/local/mysql/data/mysqld.local.err --pid-file=/usr/local/mysql/data/mysqld.local.pid --log-bin --server-id=1
```
> When enabling binary logs, the server ID is automatically set to '1'.  --(https://bugs.mysql.com/bug.php?id=56739)

Q:How can i get server_id in MYSQL?
A:SELECT @@server_id

###9.Q:如何分屏查看输出结果？
A:

```
// 用 less 分屏查看输出结果
mysql> pager less;

// 关闭分屏查看
mysql> nopager;
```

###10.Q:Scope rules for stored programs and views?
A:

###11.Q:How to view all Stored Programs and Views in MySQL?
A:

```
// 1.Stored Routines (Procedures and Functions)
> Query the ROUTINES table of the INFORMATION_SCHEMA database.
> The ROUTINES table provides information about stored routines (both procedures and functions). The ROUTINES table does not include user-defined functions (UDFs).

// 2.Triggers
mysql>SHOW TRIGGERS;

// 3.Event Scheduler
mysql>SHOW EVENTS;

// 4.Views
```
###12.Q:How to view user-defined functions (UDFs)?
A:


