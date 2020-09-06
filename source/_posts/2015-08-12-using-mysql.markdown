---
layout: post
title: "MySQL 使用笔记"
date: 2015-08-12 22:01:09 +0800
comments: true
categories: [Archives, Web Development]
keywords: MySQL, server id, notes 
description: Mysql notes.
---

###1.查看所有的用户

```
$ mysql -u user -p

As of MySQL 5.7.6, use this statement:
mysql> SELECT User, Host, HEX(authentication_string) FROM mysql.user;

Before MySQL 5.7.6, use this statement:
mysql> SELECT User, Host, Password FROM mysql.user;
```
###2.查看用户的权限

```
mysql > SHOW GRANTS FOR 'username'@'localhost';
```

###3.为用户添加操作某个数据库的权限

```
mysql > GRANT select, insert, update, delete, index, alter, create ON db2.* TO 'jeffrey'@'localhost';
mysql > GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, REFERENCES, INDEX, ALTER ON `xiao_chun`.* TO `xiaochun`@`localhost`
```

###4.Default options are read from the following files in the given order:

1. `/etc/my.cnf`
2. `/etc/mysql/my.cnf`
3. `/usr/local/mysql/etc/my.cnf`
4. `~/.my.cnf`

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

A:`SELECT @@server_id`

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

###13.mysqldump: Got error: 2013: Lost connection to MySQL server at 'reading initial communication packet', system error: 102 when trying to connect

```
DongMeiliangsMacBook-Pro:~ dongmeiliang$ mysqldump --single-transaction --databases opencart > opencart_2016_02_27_22_28.sql
mysqldump: Got error: 2013: Lost connection to MySQL server at 'reading initial communication packet', system error: 102 when trying to connect
```
A:问题的原因是我已经打开了一个mysql连接，关闭之后重试就没问题了。

###14.How to Create an Admin User Account?
A:

```
/* 1. */
mysql> CREATE USER 'monty'@'localhost' IDENTIFIED BY 'some_pass';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'localhost'
    ->     WITH GRANT OPTION;

/* 2. */
mysql> CREATE USER 'monty'@'%' IDENTIFIED BY 'some_pass';
mysql> GRANT ALL PRIVILEGES ON *.* TO 'monty'@'%'
    ->     WITH GRANT OPTION;
```

###15./usr/local/mysql/data/Mac-mini.local.err: Permission denied
A: Mysql 安装好后，启动出错，详细信息如下:  

```
Mac-mini:mysql dongmeiliang$ /usr/local/mysql/bin/mysqld_safe --user=mysql &
[3] 41906
Mac-mini:mysql dongmeiliang$ /usr/local/mysql/bin/mysqld_safe: line 647: /usr/local/mysql/data/Mac-mini.local.err: Permission denied
Logging to '/usr/local/mysql/data/Mac-mini.local.err'.
2018-04-03T08:50:51.6NZ mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/data
/usr/local/mysql/bin/mysqld_safe: line 144: /usr/local/mysql/data/Mac-mini.local.err: Permission denied
/usr/local/mysql/bin/mysqld_safe: line 198: /usr/local/mysql/data/Mac-mini.local.err: Permission denied
/usr/local/mysql/bin/mysqld_safe: line 906: /usr/local/mysql/data/Mac-mini.local.err: Permission denied
rm: /usr/local/mysql/data/Mac-mini.local.pid.shutdown: Permission denied
2018-04-03T08:50:51.6NZ mysqld_safe mysqld from pid file /usr/local/mysql/data/Mac-mini.local.pid ended
/usr/local/mysql/bin/mysqld_safe: line 144: /usr/local/mysql/data/Mac-mini.local.err: Permission denied

[3]   Done                    /usr/local/mysql/bin/mysqld_safe --user=mysql
```

从错误信息来看是权限问题。在 General Notes on Installing MySQL on macOS 中有这么一段介绍：  

>You may need (or want) to create a specific mysql user to own the MySQL directory and data. You can do this through the Directory Utility, and the mysql user should already exist. For use in single user mode, an entry for _mysql (note the underscore prefix) should already exist within the system /etc/passwd file.

~~于是来确认 mysql 账号是否存在，我们用 Directory Utility 来做这件事情。~~~~可以按 System Preferences > Users & Groups > Network Account Server join > Open Directory Utility > Directory Editor 来打开账号管理界面，~~~~在里面确实没有 mysql 账号，于是尝试创建这个账号，但是创建失败，提示账号已经存在，这就不知道是个什么鬼了。当然我们也可以使用对应的 Directory Utility 命令 dscl， 具体可参考~~ [To create a user](http://damiansheldon.github.io/blog/problems-when-use-mac.html)   

~~既然 mysql 创建不了，尝试使用其他的账号，~~~~于是把一个常用的账号加到 _mysql 组，为 /usr/local/mysql/data 赋予组写入权限，~~~~使用命令 `/usr/local/mysql/bin/mysqld_safe --user=dongmeiliang &` 成功启动。~~ 

今天把 MySQL 5.7 升级到 8.0 又遇到了启动失败的问题，似乎 mysqld_safe 这个命令在 macOS 上有点问题，我换成 mysqld 就能成功启动。  

```
$sudo mysqld --user=_mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --log-error=/usr/local/mysql/data/mysqld.local.err --pid-file=/usr/local/mysql/data/mysqld.local.pid --keyring-file-data=/usr/local/mysql/keyring/keyring --early-plugin-load=keyring_file=keyring_file.so --upgrade=FORCE &
```


