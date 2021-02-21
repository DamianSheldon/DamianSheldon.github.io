---
layout: post
title: "如何搭建一个带 Dovecot 的 Postfix 邮件服务器"
date: 2017-02-15 16:29:42 +0800
comments: true
categories: [Archivies] 
keywords: Postfix, Dovecot, E-Mail, centos 8 
description: How to set up a postfix e-mail server with dovecot? 
---

作为一个软件开发人员，我们可能需要一个自己的 VPS ，在上面可以跑我们的 side project，或者做一些实验。VPS 在运行过程中可能会遇到问题，这时我们可能希望在发生问题时能收到通知。使用邮件通知是个不错的方式。这样我们的服务挂掉了能及时处理。所以学会搭建邮件服务器还是很有价值。另外我一直觉得软件技术人员有个自己域名的邮箱很酷，所以我决定自己动手搭建一个邮件服务器。本文记录了如何搭建一个自己域名的邮件服务器，并让这个邮箱可以通过 Mac 和 iPhone 自由收发邮件(测试过 sina 和 QQ)。

## 邮件系统是怎么工作

开始之前我觉得有必要了解下邮件系统是怎么工作的，鸟哥在他的博文：[第二十二章、郵件伺服器： Postfix](http://linux.vbird.org/linux_server/0380mail.php)作了很详细的介绍，建议熟读之后再开始。

## 准备材料 

* VPS
* 域名

我服务器跑的是 CentOS 8。

<!--more-->

## 配置 DNS

### 添加 A、AAAA、MX、PTR 和 SPF 记录

我们首先需要配置 DNS， 主要是添加 A、AAAA、MX、PTR 和 SPF 记录，时间和精力允许的话建议添加 DKIM 和 DMARC 记录，否则部分邮件提供商可能拒收我们的邮件。据说 Gmail 就会验证这些记录，而 Sina 比较宽松，只验证 MX，QQ 会验证 MX 、SPF 和 PTR。

我域名是挂在 Cloudflare，完整配置如下:


类型 | 名称 | 内容 | TTL 
---- | --- | --- | --- 
A | www | 104.168.144.39 | 自动 
A | mail | 104.168.144.39 | 自动 
A | api | 104.168.144.39 | 自动 
AAAA | www | 2607:5501:3000:1008::2 | 自动 
AAAA | mail | 2607:5501:3000:1008::2 | 自动 
AAAA | api | 2607:5501:3000:1008::2 | 自动 
MX | tenneshop.com | www.tenneshop.com | 自动
PTR | 39.144.168.104.in-addr.arpa | www.tenneshop.com | 自动
PTR | 2.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.8.0.0.1.0.0.0.3.1.0.5.5.7.0.6.2.ip6.arpa | www.tenneshop.com | 自动
SPF | tenneshop.com | v=spf1 ip4:104.168.144.39 ip6:2607:5501:3000:1008::2 ~all | 自动
TXT| www | v=spf1 ip4:104.168.144.39 ip6:2607:5501:3000:1008::2 ~all | 自动

### 验证 DNS

DNS 需要几个小时才能蔓延到整个互联网，但是在你的 DNS 服务器上应该马上就会生效，你可以用 dig 来验证。

```
$ dig www.tenneshop.com @buck.ns.cloudflare.com -t A

$ dig www.tenneshop.com @buck.ns.cloudflare.com -t AAAA

$ dig tenneshop.com @buck.ns.cloudflare.com -t MX

$ dig -x 104.168.144.39

$ dig -x 2607:5501:3000:1008::2

$ dig www.tenneshop.com @buck.ns.cloudflare.com -t TXT
```

## Postfix

### 安装

我安装的 CentOS 8 默认的邮件服务程序是 sendmail，我们先停掉它，然后安装 postfix。

```
$ sudo systemctl disable sendmail.service
$ sudo systemctl stop sendmail.service
# 确认是否已安装  postfix
$ rpm -q postfix
$ sudo dnf search postfix
$ sudo dnf install postfix.x86
```

### 配置

Postfix 有两个主要配置文件 `/etc/postfix/main.cf` 和 `/etc/postfix/master.cf`, main.cf 指定配置项；master.cf 指定 postfix 要运行哪些服务。

配置的主要工作是设置 main.cf 这个文件里的配置项。

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
    mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain

```

* 配置信任的终端

```
/etc/postfix/main.cf:
mynetworks = 127.0.0.0/8, hash:/etc/postfix/access 
```

配置完我们需要执行命令：`sudo postmap hash:/etc/postfix/access`。

* 设置邮件别名，邮件别名对我们有大用处，首先我们可以用它来实现一般帐号接收 root 信件，其次我们还可以用它实现将用户的信件发送一份到对应外域邮箱。

```
/etc/aliases:

root: root,meiliang
yourname: meiliang,dongmeilianghy@sina.com
```

更新完了`/etc/aliases`，还需要运行命令去生效:

```
$ newaliases
```

我们可以使用 `sudo postconf -n` 来看下得到的完整配置:

```
alias_database = hash:/etc/aliases
alias_maps = hash:/etc/aliases
command_directory = /usr/sbin
compatibility_level = 2
daemon_directory = /usr/libexec/postfix
data_directory = /var/lib/postfix
debug_peer_level = 2
debugger_command = PATH=/bin:/usr/bin:/usr/local/bin:/usr/X11R6/bin ddd $daemon_directory/$process_name $process_id & sleep 5
html_directory = no
inet_interfaces = all
inet_protocols = all
mail_owner = postfix
mailq_path = /usr/bin/mailq.postfix
manpage_directory = /usr/share/man
meta_directory = /etc/postfix
mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain, hwsrv-773010.hostwindsdns.com
mydomain = tenneshop.com
myhostname = www.tenneshop.com
mynetworks = 127.0.0.0/8, hash:/etc/postfix/access
myorigin = $mydomain
newaliases_path = /usr/bin/newaliases.postfix
queue_directory = /var/spool/postfix
readme_directory = /usr/share/doc/postfix/README_FILES
relay_domains = $mydestination
sample_directory = /usr/share/doc/postfix/samples
sendmail_path = /usr/sbin/sendmail.postfix
setgid_group = postdrop
shlib_directory = /usr/lib64/postfix
smtp_tls_CAfile = /etc/pki/tls/certs/ca-bundle.crt
smtp_tls_CApath = /etc/pki/tls/certs
smtp_tls_security_level = may
smtpd_tls_cert_file = /etc/pki/tls/certs/postfix.pem
smtpd_tls_key_file = /etc/pki/tls/private/postfix.key
smtpd_tls_security_level = may
unknown_local_recipient_reject_code = 550
```

* 防火墙设置

因为整个 MTA 主要透过 SMTP (port 25) 进行信件传送的任务，因此，针对 postfix 来说，只要放行 port 25 即可!

```
$sudo iptables -A INPUT -p TCP -i eth0 --dport  25  --sport 1024:65534 -j ACCEPT
```

一切准备就绪之后，我们使用 `sudo systemctl start postfix.service` 来启动 postfix, 可以用 `systemctl status postfix.service` 确认是否启动成功，有问题的话，根据错误信息对应解决。

### 测试

我们可以使用 mail 命令来发送邮件做测试:

```
$ mail -r foo@tenneshop.com bar foobar@example.com
Subject: Does postfix works?
Does postfix works?
<Ctrl+D>
```

然后我们可以查看 postfix 的日志确认邮件是否能正常投递，有问题的话也可以对应解决:

```
// mail log
$ sudo tail -f /var/log/maillog
```

邮件正常投递出去之后，我们可以使用 mail 命令来查看本地邮箱的邮件，外域邮箱的话则使用对应的客户端查看确认。  

发送邮件功能测试完，接下来就是测试接收邮件，我们用外域邮箱给新搭建邮件服务器上的用户发送一封邮件，然后登录 VPS 确认他的本地邮箱是否收到了发送的信件。  

一切正常的话，我们就有了一个可以工作的邮件服务器了，她能满足我们绝大多数需求了，只是我们现在如果要发送邮件的话得登录 VPS，然后使用 mail 命令行来操作。如果我们发送邮件需求不强烈的话，架设工作可以到此结束了；如果我们还想用 iPhone 和 Mac 的 Mail App 来收发邮件的话，那还需要做些配置。

要想用 iPhone 和 Mac 的 Mail App 来收邮件的话，我们得架设 MRA。哪么使用哪个 MRA 服务器呢？ dovecot 是个不错的选择。

发邮件话则需要开放 SMTP 的身份认证，目前 postfix 支持 cyrus 和 dovecot 两种 SASL 认证，既然收邮件需要用到 dovecot，那发邮件也用它的 SASL 认证好了，这样可以少装些软件。

## Dovecot

接下来，我们先安装 Dovecot，然后让系统用户可以通过它收邮件，收邮件配置好，我们再配置 postfix 使用 dovecot SASL。

### 安装

Dovecot 支持 imap 和 pop3，由于 pop3 会把服务器上的邮件删除掉，所以这里我使用 imap,

```
$ dnf search dovecot
$ sudo dnf install dovecot.x86_64
```

### 配置

个人觉得 Dovecot 的文档比 postfix 对新手友好，我们可以参考它的 [Dovecot installation](wiki.dovecot.org)，配置也尽量循序渐进，先使用系统用户明文认证，可以工作之后再开启 TLS，TLS 也配好后再考虑加入虚拟用户，这样把问题分解，难度就降低了，配置项也集中在一个逻辑块上，容易配好。

* 检查邮件投递位置

CentOS 8 默认的邮件投递位置是 `~/spool/mail`， 更新到配置文件中:

```
/etc/dovecot/conf.d/10-mail.conf:
mail_location = mail_location = mbox:~/spool/mail:INBOX=/var/spool/mail/%u
```

* 配置 Dovecot

```
// 使用命令找到 Dovecot 配置文件的位置:
$ doveconf -n | head -n1
/etc/dovecot/dovecot.conf

// 开启 imap
protocols = imap

// 明文认证 Plaintext Authentication
To allow any Authentication without SSL, disable SSL in the conf.d/10-ssl.conf file.
/etc/dovecot/conf.d/10-ssl.conf:
ssl = no

Until SSL is configured, allow plaintext authentication in the conf.d/10-auth.conf file. You probably want to switch this back to "yes" afterward.
/etc/dovecot/conf.d/10-auth.conf:
disable_plaintext_auth = no
```

* 启动 Dovecot

```
// Start Dovecot
$ sudo systemctl start dovecot.service

// Stop Dovecot
$sudo systemctl stop dovecot.service

// Restart 
$sudo systemctl restart dovecot.service

// Verify
$ sudo systemctl status dovecot.service
```

* 配置防火墙

```
iptables -A INPUT -p TCP -i eth0 --dport 143  --sport 1024:65534 -j ACCEPT
```

* 测试一切是否都正常工作

我们可以参考 Dovecot [Testing that everything works](https://wiki.dovecot.org/TestInstallation) 提供的指令来测试 Dovecot 是否正常工作。

### 检查它是否在监听

```
$ telnet localhost 143
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE STARTTLS AUTH=PLAIN] Dovecot ready.

If you got "connection refused", make sure that Dovecot is configured to serve the imap protocol and listening on the expected interfaces/addresses. The simplest way to do that would be using doveconf(1):
$ sudo doveconf protocols listen
protocols = imap
listen = *, ::
```

### 接下来检查它是否在远程主机上也能工作:

```
$ telnet imap.example.com 143
Trying 1.2.3.4...
Connected to imap.example.com.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE STARTTLS AUTH=PLAIN] Dovecot ready.
```

### 检查是否允许登录

```
% telnet localhost 143
a login "username" "password"
a OK Logged in.
```

### 检查是否允许远程登录

```
$ telnet imap.example.com 143
a login "username" "password"

```

### 检查是否找到收件箱

```
b select inbox
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 1 EXISTS
* 1 RECENT
* OK [UIDVALIDITY 1106186941] UIDs valid
* OK [UIDNEXT 2] Predicted next UID
b OK [READ-WRITE] Select completed.
```

在这一步我遇到了一个权限问题，详细的错误日志如下:  

```
imap(meiliang)<3989594><nOvNDc+7rpIAAAAAAAAAAAAAAAAAAAAB>: Error: fchown(/home/meiliang/spool/mail/.imap/INBOX, group=12(mail)) failed: Operation not permitted (egid=1000(meiliang), group based on /var/spool/mail/meiliang - see http://wiki2.dovecot.org/Errors/ChgrpNoPerm)
```

Dovecot 的 wiki 详细地解释了这个问题：  

> This means that Dovecot tried to copy /var/mail/user file's group (mail) to the index file directory it was creating (/home/user/mail/.imap/INBOX), but the process didn't belong to the mail group, so it failed. This is important for preserving access permissions with shared mailboxes. Group copying is done only when it actually changes the access permissions; for example with 0600 or 0666 mode the group doesn't matter at all, but with 0660 or 0640 it does.


问题的原因是 dovecot 尝试把邮件复制到它创建的索引文件目录时，如果文件的权限是 0660 或 0640，它会尝试保留原来的 group ，但由于 dovecot 不在 mail 这个 group，所以没有权限，操作就失败了。 Dovecot 给出了两种解决方案:  

> To solve this problem you can do only one of two things:
> 	a.	If the group doesn't actually matter, change the permissions so that the group isn't copied (e.g. chmod 0600 /var/mail/*, see MailLocation/mbox) 
> 	b.	Give the mail process access to the group (e.g. mail_access_groups=mail setting). However, this is dangerous. It allows users with shell access to read other users' INBOXes.  


另外 [MailLocation/mbox](https://wiki2.dovecot.org/MailLocation/mbox) 中也说明了这个问题: 

> `/var/mail/*` permissions

> In some systems the /var/mail/$USER files have 0660 mode permissions. This causes Dovecot to try to preserve the file's group, and if it doesn't have permissions to do so, it'll fail with an error:

```
imap(user): Error: chown(/home/user/mail/.imap/INBOX, -1, 12(mail)) failed: Operation not permitted (egid=1000(user), group based on /var/mail/user)
```

> There is rarely any real need for the files to have 0660 mode, so the best solution for this problem is to just change the mode to 0600:

```
chmod 0600 /var/mail/*
```

综合两处信息，我们可以知道将 `/var/spool/mail/*` 的权限改成 0600 是最佳解决方案。

### 优雅地退出

```
e logout
* BYE Logging out
e OK Logout completed.

```

收邮件配好后，我们就来配置 postfix 的 SASL。

## 使能 postfix 的 SASL

我们重点参考 postfix [SASL Authentication](http://www.postfix.org/SASL_README.html) 的 Configuring SASL authentication in the Postfix SMTP server。

我们先确认下 postfix SASL 实现的支持情况:

```
$ postconf -a
cyrus
dovecot
```

* 配置 Dovecot SASL

Dovecot 独立地去认证它的 POP/IMAP 终端，Postfix 使用 Dovecot SASL 时会复用这部分配置。

Postfix 到 Dovecot SASL 的通信有两种途径:UNIX-domain socket or TCP socket。从更好的安全性角度出发，我们选择使用 UNIX-domain socket.

```
/etc/dovecot/conf.d/10-master.conf:
service auth {
    ...
    unix_listener /var/spool/postfix/private/auth {
        mode = 0660
        # Aussuming the default postfix user and group
        user = postfix
        group = postfix
    }
}

/etc/dovecot/conf.d/10-auth.conf
auth_mechanisms = plain login
```

* 在 Postfix SMTP 服务器中启用 SASL 认证和授权

默认 postfix 是使用 Cyrus SASL 实现，我们需要改成 dovecot

```
/etc/postfix/main.cf:
smtpd_sasl_type = dovecot
```

指定 Postfix 怎么和 Dovecot 认证服务器通信, 这里我们使用 UNIX-domain socket:

```
/etc/postfix/main.cf:
smtpd_sasl_path = private/auth
```

###开启 SASL 认证:

```
/etc/postfix/main.cf:
smtpd_sasl_auth_enable = yes
```

重启下 postfix, 使用 SMTP 命令验证配置是否生效:

```
% telnet server.example.com 25
...
220 server.example.com ESMTP Postfix
EHLO client.example.com
250-server.example.com
250-PIPELINING
250-SIZE 10240000
250-AUTH DIGEST-MD5 PLAIN CRAM-MD5
...
```

配置 Postfix SMTP 服务器的安全原则：

未加密的 SMTP 会话

```
/etc/postfix/main.cf:
smtpd_sasl_security_options = noanonymous
```

加密的 SMTP 会话

```
/etc/postfix/main.cf:
smtpd_sasl_tls_security_options = $smtpd_sasl_security_options
```

要在建立了 TLS 加密的会话后才提供 SASL 认证，请指定以下内容:

```
/etc/postfix/main.cf:
smtpd_tls_auth_only = yes
```

在 Postfix SMTP 服务器中启用 SASL 授权

客户端通过SASL认证后，Postfix SMTP 服务器会决定远程 SMTP 客户端的授权内容。可能的 SMTP 客户端授权的例子有:

* 向远程收件人发送消息。
* 在 MAIL FROM 命令中使用特定的信封发件人.

这些权限默认不启用。

* 邮件中继授权

使用 permit_sasl_authenticated，Postfix SMTP 服务器可以允许 SASL 认证的 SMTP 客户发送邮件到远程目的地。例子:

```
/etc/postfix/main.cf:
smtpd_relay_restrictions =
permit_mynetworks
permit_sasl_authenticated
reject_unauth_destination
```

* 信封发件人地址授权

默认情况下，SMTP 客户端可以在 MAIL FROM 命令中指定任何信封发件人地址。这是因为 Postfix SMTP 服务器只知道远程SMTP客户端的主机名和 IP 地址，但不知道控制远程 SMTP 客户端的用户。

这在 SMTP 客户端使用 SASL 认证的那一刻就发生了变化，现在，Postfix SMTP 服务器知道谁是发件人了。给定一个信封发件人地址和 SASL 登录名的表格，Postfix SMTP 服务器可以决定是否允许 SASL 认证的客户端使用特定的信封发件人地址:

```
/etc/postfix/main.cf:
smtpd_sender_login_maps = hash:/etc/postfix/controlled_envelope_senders

smtpd_recipient_restrictions =
...
reject_sender_login_mismatch
permit_sasl_authenticated
...
```

controlled_envelope_senders 表指定了发件人信封地址和拥有该地址的 SASL 登录名之间的绑定:

```
/etc/postfix/controlled_envelope_senders
# envelope sender           owners (SASL login names)
meiliang@tenneshop.com      meiliang
```

配置完记得执行 `sudo postmap hash:/etc/postfix/controlled_envelope_senders`。

这样一来，如果 smtpd_sender_login_maps 没有指定 SMTP 客户端的登录名作为该地址的所有者，上面的 reject_sender_login_mismatch 限制将拒绝 MAIL FROM 命令中的发件人地址。

* 在Postfix SMTP服务器中测试SASL认证

要测试服务器端，连接（例如，用telnet）到Postfix的SMTP服务器端口，你应该能够有一个对话，如下所示。

```
% telnet server.example.com 25
...
220 server.example.com ESMTP Postfix
EHLO client.example.com
250-server.example.com
250-PIPELINING
250-SIZE 10240000
250-ETRN
250-AUTH DIGEST-MD5 PLAIN CRAM-MD5
250 8BITMIME
AUTH PLAIN AHRlc3QAdGVzdHBhc3M=
235 Authentication successful
```

要通过 TLS 加密的连接进行测试，请使用 openssl s_client 代替 telnet:

```
% openssl s_client -connect server.example.com:25 -starttls smtp
...
220 server.example.com ESMTP Postfix
EHLO client.example.com
...see above example for more...
```

不使用 `AHRlc3QAdGVzdHBhc3M=`，而是指定 base64 编码形式的`\0username\0password`(其中/0为空字节)。上面的例子是针对一个名为 "test "的用户，密码为 "testpass"。

这里我们要注意一下 PLAIN 认证方法用户名和密码的提交格式，Dovecot 对它有详细的介绍:

> The PLAIN mechanism's authentication format is:` <authorization ID> NUL <authentication ID> NUL <password>`. Authorization ID is the username who you want to log in as, and authentication ID is the username whose password you're giving. If you're not planning on doing a master user login, you can either set both of these fields to the same username, or leave the authorization ID empty.

因为我们不打算做所谓的 master user login，所以我们可以用 `NUL <authentication ID> NUL <password>` 的格式，也就是 postfix 所说的 `\0username\0password`，postfix 和 dovecot 都给我们提供好几种 base64 编码的方法，我在这里踩坑了，由于我的密码含有数字，使用 `% echo -ne '\000username\000password' | openssl base64
` 生成的 base64 编码不对，导致认证总是失败。

由于 LOGIN 命令是先提交base64编码的用户名，然后提交base64编码的密码，于是借助它才找出是转义特殊字符失败。

```
auth login 
334 VXNlcm5hbWU6
c2VydmljZUBoZWVwLmNx
334 UGFzc3dvcmQ6
xxxxxxxx
235 Authentication successful
```

所以我推荐 `% printf '\0%s\0%s' 'username' 'password' | openssl base64` 来生成 base64 编码。

一切都正常工作之后，我们就可以在 Mac 和 iPhone 上配置 Mail 账号来收发邮件了，只是我们目前是使用明文且未加密的会话通道，所以并不安全，接下来就是启用 TLS。

## 启用 TLS (TODO)

## 虚拟用户(TODO)

## 附录

SMTP 命令列表如下：

HELO

客户端为标识自己的身份而发送的命令（通常带域名）

EHLO

使服务器可以表明自己支持扩展简单邮件传输协议 (ESMTP) 命令。

MAIL FROM

标识邮件的发件人；以 MAIL FROM: 的形式使用。

RCPT TO

标识邮件的收件人；以 RCPT TO: 的形式使用。

TURN

允许客户端和服务器交换角色，并在相反的方向发送邮件，而不必建立新的连接。

ATRN

ATRN (Authenticated TURN) 命令可以选择将一个或多个域作为参数。如果该会话已通过身份验证，则ATRN 命令一定会被拒绝。

SIZE

提供一种使 SMTP 服务器可以指出所支持的最大邮件大小的机制。兼容的服务器必须提供大小范围，以指出可以接受的最大邮件大小。客户端发送的邮件不应大于服务器所指出的这一大小。

ETRN

SMTP 的扩展。SMTP 服务器可以发送 ETRN 以请求另一台服务器发送它所拥有的任何电子邮件。

PIPELINING

提供发送命令流（而无需在每个命令之后都等待响应）的能力。

CHUNKING

替换 DATA 命令的 ESMTP 命令。该命令使 SMTP 主机不必持续地扫描数据的末尾，它发送带参数的 BDAT 命令，该参数包含邮件的总字节数。接收方服务器计算邮件的字节数，如果邮件大小等于 BDAT 命令发送的值时，则该服务器假定它收到了全部的邮件数据。

DATA

客户端发送的、用于启动邮件内容传输的命令。

DSN

启用传递状态通知的 ESMTP 命令。

RSET

使整个邮件的处理无效，并重置缓冲区。

VRFY

确认在邮件传递过程中可以使用邮箱；例如，vrfy ted 确认在本地服务器上驻留 Ted 的邮箱。该命令在 Exchange 实现中默认关闭。

HELP

返回 SMTP 服务所支持的命令列表。

QUIT

终止会话。


##Reference:

* [How To Set Up a Postfix E-Mail Server with Dovecot](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-postfix-e-mail-server-with-dovecot#postfix)  
* [Postfix](http://www.postfix.org/)  
* [Dovecot](http://wiki.dovecot.org/)  
* [Linux mail command examples – send mails from command line](http://www.binarytides.com/linux-mail-command-examples/)  
* [SMTP的相关命令](http://www.cnblogs.com/cocowool/archive/2012/03/14/2395390.html)  
* [IMAP 101: Manual IMAP Sessions](https://www.atmail.com/blog/imap-101-manual-imap-sessions/)  

##修改记录

* 2021/02/20 迭代重写，将模糊的地方讲清楚，时间关系暂未启用 TLS 及支持虚拟用户
* 2017/02/15 第一次完成


