---
layout: post
title: "OpenLDAP 学习笔记"
date: 2018-07-10 21:58:07 +0800
comments: true
categories: [Archives]
keywords: OpenLDAP, LDAP, Mac, slapd, ldapsearch
description: OpenLDAP learning notes about how to use it on Mac.
---

最近业余时间在自学 Java，接触到 JNDI，继而牵扯出 LDAP。在倒腾过程中感觉是个复杂的主题，决定做个简要的笔记，梳理下思路，也方便日后忘记时容易拾起。  

OpenLDAP 是 LDAP 协议的一个开源实现。LDAP 服务器本质上是一个为只读访问而优化的非关系型数据库。它主要用做地址簿查询（如 email 客户端）或对各种服务访问做后台认证以及用户数据权限管控。（例如，访问 Samba 时，LDAP 可以起到域控制器的作用；或者 Linux 系统认证 时代替 /etc/passwd 的作用。）  

以 slap 形状的命令如: slapacl, slapadd 等是服务端工具；以 ldap 开头的命令如: ldapadd, ldapcompare 等是客户端命令工具。

##安装

Mac 内置了 OpenLDAP 软件包，所以可以直接使用。

##配置

###服务端
服务端支持两种配置方法:动态运行时配置引擎和老式的 slapd.conf 文件。这里介绍通过老式的 slapd.conf 文件转换成动态运行时配置引擎来配置服务器的方法。  

服务器的配置文件位于 `/etc/openldap/slapd.conf`。在 Mac 上第一次配置时该文件还不存在，但有一个 slapd.conf.default 文件，从文件名可知，这应该是一个默认配置文件，所以我们可以在它基础上来配置。  

```
$ sudo cp /etc/openldap/slapd.conf.default /etc/openldap/slapd.conf
```

需要编辑后缀和 rootdn。典型的后缀通常是你所用的域名，但这并非强制要求，而是依赖于你如何使用你的目录。下例中以 tenneshop 做为域名，tld 为 com，rootdn 则是 LDAP 管理员的名字（这里用 Manager）。
<!--more-->
```
suffix     "dc=tenneshop,dc=com"
rootdn     "cn=Manager,dc=example,dc=com"
```

现在删除默认 root 口令并创建一个强口令：

```
$slappasswd -s yourfavoritepassword

// /etc/openldap/slapd.conf
rootpw  {SSHA}Q0vdi3/5Hw+EhDrFAbvEEszq1Xf4YSyy
```

在 slapd.conf 头部添加一些 schemas:

```
 include     /private/etc/openldap/schema/core.schema
 include         /etc/openldap/schema/cosine.schema
 include         /etc/openldap/schema/inetorgperson.schema
 include         /etc/openldap/schema/nis.schema
 #include         /etc/openldap/schema/samba.schema
 include     /private/etc/openldap/schema/java.schema
```

可能需要在 slapd.conf 底部加入一些常用的 indexes:

```
index   uid             pres,eq
index   mail            pres,sub,eq
index   cn              pres,sub,eq
index   sn              pres,sub,eq
index   dc              eq
```

现在准备数据目录，需要重命名配置文件：

```
$sudo mkdir -p /private/var/db/openldap/openldap-data
$sudo cp /etc/openldap/DB_CONFIG.example /private/var/db/openldap/openldap-data/DB_CONFIG
```

将 slapd.conf 中的改动应用到 `/etc/openldap/slapd.d/`(第一次配置时可能需要创建该目录)，*需要先删除老配置*：

```
$sudo rm -rf /etc/openldap/slapd.d/*

```

用下面命令生成配置文件:

```
$sudo slaptest -f /etc/openldap/slapd.conf -F /etc/openldap/slapd.d/
```

每次修改 slapd.conf 后，都需要执行上面命令。检查有没有问题，可以忽略 "bdb_monitor_db_open: monitoring disabled; configure monitor database to enable". 如果提示数据库不存在，可以先使用命令来检查配置文件:

```
$sudo slaptest -f /etc/openldap/slapd.conf -F /etc/openldap/slapd.d/ -u 
```

确认配置文件没有语法错误之后，可以通过启动 slap 来创建数据库:

```
$sudo /usr/libexec/slapd -F /etc/openldap/slapd.d
```

最终完整的配置文件如下:

```
#
# See slapd.conf(5) for details on configuration options.
# This file should NOT be world readable.
#
include		/private/etc/openldap/schema/core.schema
include         /etc/openldap/schema/cosine.schema
include         /etc/openldap/schema/inetorgperson.schema
include         /etc/openldap/schema/nis.schema
#include         /etc/openldap/schema/samba.schema
include		/private/etc/openldap/schema/java.schema

# Define global ACLs to disable default read access.

# Do not enable referrals until AFTER you have a working directory
# service AND an understanding of referrals.
#referral	ldap://root.openldap.org

pidfile		/private/var/db/openldap/run/slapd.pid
argsfile	/private/var/db/openldap/run/slapd.args

# Load dynamic backend modules:
# modulepath	/usr/libexec/openldap
# moduleload	back_bdb.la
# moduleload	back_hdb.la
# moduleload	back_ldap.la

# Sample security restrictions
#	Require integrity protection (prevent hijacking)
#	Require 112-bit (3DES or better) encryption for updates
#	Require 63-bit encryption for simple bind
# security ssf=1 update_ssf=112 simple_bind=64

# Sample access control policy:
#	Root DSE: allow anyone to read it
#	Subschema (sub)entry DSE: allow anyone to read it
#	Other DSEs:
#		Allow self write access
#		Allow authenticated users read access
#		Allow anonymous users to authenticate
#	Directives needed to implement policy:
# access to dn.base="" by * read
# access to dn.base="cn=Subschema" by * read
# access to *
#	by self write
#	by users read
#	by anonymous auth
#
# if no access controls are present, the default policy
# allows anyone and everyone to read anything but restricts
# updates to rootdn.  (e.g., "access to * by * read")
#
# rootdn can always read and write EVERYTHING!

#######################################################################
# BDB database definitions
#######################################################################

database	bdb
suffix		"dc=tenneshop,dc=com"
rootdn		"cn=Manager,dc=tenneshop,dc=com"
# Cleartext passwords, especially for the rootdn, should
# be avoid.  See slappasswd(8) and slapd.conf(5) for details.
# Use of strong authentication encouraged.
#rootpw		secret
#The hash was generated from password secret using the command slappasswd -s secret
rootpw	{SSHA}Q0vdi3/5Hw+EhDrFAbvEEszq1Xf4YSyy
# The database directory MUST exist prior to running slapd AND
# should only be accessible by the slapd and slap tools.
# Mode 700 recommended.
directory	/private/var/db/openldap/openldap-data
# Indices to maintain
index	objectClass	eq

# Some common indexes
index   uid             pres,eq
index   mail            pres,sub,eq
index   cn              pres,sub,eq
index   sn              pres,sub,eq
index   dc              eq
```

###客户端
客户的配置文件位于 /etc/openldap/ldap.conf.

这个配置很简单，只需要将BASE 设置为服务器的前缀，将 URI 设置为服务器的地址:

```
BASE            dc=tenneshop,dc=com
URI             ldap://localhost
```

最终完整的配置文件如下:

```
#
# LDAP Defaults
#

# See ldap.conf(5) for details
# This file should be world readable but not world writable.

#BASE	dc=example,dc=com
#URI	ldap://ldap.example.com ldap://ldap-master.example.com:666
BASE	dc=tenneshop,dc=com
URI	ldap://localhost

#SIZELIMIT	12
#TIMELIMIT	15
#DEREF		never
#TLS_REQCERT	demand
TLS_REQCERT allow
```

###创建初始项

配置好客户端后，创建根项和 root 角色项：
```
dn: dc=tenneshop,dc=com
objectClass: dcObject
objectClass: organization
dc: tenneshop
o: Tenneshop
description: Tenneshop directory

dn: cn=Manager,dc=tenneshop,dc=com
objectClass: organizationalRole
cn: Manager
description: Directory Manager
```

将上述内容保存在文件 `/etc/openldap/root.ldif` 中，然后使用命令:

```
$ldapadd -x -D 'cn=Manager,dc=tenneshop,dc=com' -W -f /etc/openldap/root.ldif
```
添加到目录服务中。

###测试安装好的系统

运行下面命令:

```
$ ldapsearch -x '(objectclass=*)'
# extended LDIF
#
# LDAPv3
# base <dc=tenneshop,dc=com> (default) with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# tenneshop.com
dn: dc=tenneshop,dc=com
objectClass: dcObject
objectClass: organization
dc: tenneshop
o: Tenneshop
description: Tenneshop directory

# Manager, tenneshop.com
dn: cn=Manager,dc=tenneshop,dc=com
objectClass: organizationalRole
cn: Manager
description: Directory Manager

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
```

或认证为 rootdn (将 -x 替换为 -D <user> -W), 用上面配置的例子的话：

```
$ ldapsearch -D "cn=Manager,dc=tenneshop,dc=com" -W '(objectclass=*)'
Enter LDAP Password:
# extended LDIF
#
# LDAPv3
# base <dc=tenneshop,dc=com> (default) with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# tenneshop.com
dn: dc=tenneshop,dc=com
objectClass: dcObject
objectClass: organization
dc: tenneshop
o: Tenneshop
description: Tenneshop directory

# Manager, tenneshop.com
dn: cn=Manager,dc=tenneshop,dc=com
objectClass: organizationalRole
cn: Manager
description: Directory Manager

# search result
search: 2
result: 0 Success

# numResponses: 3
# numEntries: 2
```

###遇到的问题
####1.additional info: objectClass: value #0 invalid per syntax

```
$ ldapadd -x -D 'cn=Manager,dc=tenneshop,dc=com' -W -f /etc/openldap/root.ldif
Enter LDAP Password:
ldapadd: attributeDescription "dn": (possible missing newline after line 8, entry "dc=tenneshop,dc=com"?)
adding new entry "dc=tenneshop,dc=com"
ldap_add: Invalid syntax (21)
	additional info: objectClass: value #0 invalid per syntax
```

A:导入的数据每行结尾含有空格所致，去掉数据每行结尾的空格。

Reference:[OpenLDAP报错: additional info: objectClass: value #0 invalid per syntax](http://www.what21.com/sys/ldap_3_1483460096406.html)  

##Reference

* OpenLDAP Software 2.4 Administrator's Guide
* [openLDAP](https://wiki.archlinux.org/index.php/OpenLDAP_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#.E5.88.9B.E5.BB.BA.E5.88.9D.E5.A7.8B.E9.A1.B9)  
* [使用OpenLDAP实现集中式认证](https://wiki.gentoo.org/wiki/Centralized_authentication_using_OpenLDAP/zh#Configuring_the_OpenLDAP_client_tools)  
* [User Management in OpenLDAP](https://techhelplist.com/tech-tutorials/34-openldap/48-user-management-in-openldap)  



