---
layout: post
title: "如何搭建一个带 Dovecot 的 Postfix 邮件服务器"
date: 2017-02-15 16:29:42 +0800
comments: true
categories: [Archivies] 
keywords: Postfix, Dovecot, E-Mail, debian, jessie 
discription: 
---

## Prerequisites

* VPS
* 域名

## 配置 DNS

### 添加 MX 记录

添加从域名到 VPS IP 的 MX 记录，它指定了负责处理这个域名邮件收发的服务器。

```
Type    Hostname        Value         TTL
MX      tenneshop.com   1.2.3.4.      1800
```

### 验证 DNS

DNS 需要几个小时才能蔓延到整个互联网，但是在你的 DNS 服务器上几分钟之后就会生效，你可以用 dig 和 host 来验证。

```
[root@yourbase] ~# dig MX mydomain.com +short @ns1.digitalocean.com
50 mail.mydomain.com.
[root@yourbase] ~# host mail.mydomain.com ns1.digitalocean.com
Using domain server:
Name: ns1.digitalocean.com
Address: 198.199.120.125#53
Aliases:

mail.mydomain.com has address 82.196.9.119
```
<!--more-->
## how does every thing works?

## Postfix

### 安装

Debian 默认的邮件服务程序是 exim，我们需要卸载掉它。

```
$ sudo aptitude remove exim4 && aptitude install postfix && postfix stop
```

### 配置

Postfix 有两个主要配置文件 /etc/postfix/main.cf 和 /etc/postfix/master.cf, main.cf 指定配置项；master.cf 指定 postfix 要运行哪些服务。

配置的主要工作自然是编辑 main.cf 这个文件里的配置项，安全起见我们可以先把默认的配置文件做个备份:

```
$ cp /etc/postfix/main.cf /etc/postfix/main.cf.orig
```

* 配置发送邮件使用的域名;

我们可能希望别人收到我们的邮件时显示的收件人是user@example.com这种形式而不是user@mail.example.com,这可以通过设置myorigin来实现:

```
/etc/postfix/main.cf
myorigin = $mydomain
```
* 希望收到哪些域名的邮件

```
domain-wide mail server
/etc/postfix/main.cf:
    mydestination = $myhostname localhost.$mydomain localhost $mydomain

```

* 配置哪些终端可以中继邮件

```
/etc/postfix/main.cf:
mynetworks_style = host
mynetworks = 127.0.0.0/8 168.100.189.2/32
```

* 配置邮件服务器的基本账号，例如 postmaster, 这是必须要有的账号，这样别人才能反馈邮件投递的问题。

```
/etc/aliases:
mailer-daemon: postmaster
postmaster: root
nobody: root
hostmaster: root
usenet: root
news: root
webmaster: root
www: root
ftp: root
abuse: root
root: yourname
```

更新完了/etc/aliases，还需要运行命令去生效:

```
$ newaliases
```

* 配置邮件用户账号


### 测试

## Dovecot

##Reference:
[How To Set Up a Postfix E-Mail Server with Dovecot](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-postfix-e-mail-server-with-dovecot#postfix)  
[How To Install and Configure Postfix on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-on-ubuntu-16-04#conclusion)  
[Linux mail command examples – send mails from command line](http://www.binarytides.com/linux-mail-command-examples/)
