---
layout: post
title: "如何搭建一个带 Dovecot 的 Postfix 邮件服务器"
date: 2017-02-15 16:29:42 +0800
comments: true
categories: [Archivies] 
keywords: Postfix, Dovecot, E-Mail, debian, jessie 
discription: 
---
不知道从什么时候开始，也不知道是什么原因，我一直觉得有个自己域名的邮箱很酷，所以我决定自己动手搭建一个邮件服务器。本文记录了我搭建一个自己域名的邮件服务器，并让这个邮箱可以通过 Mac 和 iPhone 自由收发邮件的过程。

## 准备材料 

* VPS
* 域名

VPS 我用的 DigitalOcean，打个小广告，如果你也想买个他们家的VPS,可以用我的优惠码：https://m.do.co/c/537dc7bd8a78

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
## 邮件系统是怎么工作

开始之前我觉得有必要了解下邮件系统是怎么工作的，Dovecot 上有篇文章介绍的很好，值得看下：[Overview of how everything works together](http://wiki.dovecot.org/MailServerOverview),
我简单地说一下我的理解，先看发邮件，以使用 Mac 的 Mail 为例，用户用 Mail 写了封邮件，邮件包含收件人地址和信件内容，然后点击发送，Mail 在这里就是 MUA(Mail User Agent), MUA 把邮件发送到 Mail Server, Mail Server 称之为 MTA(Mail Transfer Agent), MTA 根据收件人地址去查询 DNS 记录，然后把邮件发送到目标MTA, 目标MTA则负责把邮件传送给MDA(Mail Delivery Agent), MDA则负责把邮件存储到邮件服务器上。

再看收邮件，用户打开 Mail 之后会触发收件操作，通常是通过 IMAP 或 POP3 协议去获取 MDA 存储在 Mail Server 上的邮件。

我们这里 MTA 用的 Postfix, IMAP/POP3 Server 用的 Dovecot.

## Postfix

### 安装

Debian 默认的邮件服务程序是 exim，我们需要卸载它。

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
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 
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

Postfix 使用数据库文件来控制权限：

```
/etc/postfix/main.cf:
    virtual_alias_maps = hash:/etc/postfix/virtual

/etc/postfix/virtual:
// Map Mail Addresses to Linux Accounts
contact@example.com sammy
admin@example.com sammy
```

我们可以通过如下命令来让映射生效:

```
$ sudo postmap /etc/postfix/virtual
```

* 配置邮件的存储文件格式

> There are two primary storage options of mail in the *NIX world: mbox and Maildir. mbox stores multiple messages - sometimes hundreds or thousands of messages - in a single file. Maildir stores each message a separate file.

Postfix 默认是 mbox 的格式，这里我们决定使用 Maildir 的格式，可以用 postconf 命令来配置:

```
sudo postconf -e 'home_mailbox= Maildir/'
```

更新环境变量 MAIL 的值：

```
echo 'export MAIL=~/Maildir' | sudo tee -a /etc/bash.bashrc | sudo tee -a /etc/profile.d/mail.sh
```

让变量在当前的会话中生效：

```
$ source /etc/profile.d/mail.sh
```

### 测试

本地测试之前需要初始化 Maildir 的目录结构，在我们 home 目录中创建 Maildir 目录结构最简单的方法是向我们自己发一封邮件，可以使用 mail 命令:

```
// 安装 mail 命令
$ sudo apt-get install mailutils

// 发送邮件
$ echo 'init' | mail -s 'init' sammy

// 确认 Maildir 目录结构初始化好了
$ ls -R ~/Maildir

// 输出的结果： 
/home/sammy/Maildir/:
cur  new  tmp

/home/sammy/Maildir/cur:

/home/sammy/Maildir/new:
1463177269.Vfd01I40e4dM691221.mail.example.com

/home/sammy/Maildir/tmp:
```

使用 mail 命令发送邮件:

```
$ mail -s "Hello World" someone@example.com
Cc: 
Hi Peter
How are you
I am fine
Good Bye
<Ctrl+D>
```

使用 mail 命令查看邮件：

```
$ mail
Heirloom mailx version 12.5 6/20/10.  Type ? for help.
"/var/mail/enlightened": 7 messages 3 unread
O  1 Enlightened        Sat Dec  6 11:33   21/658   This is the subject
O  2 Enlightened        Sat Dec  6 11:34  773/25549 This is the subject
O  3 Enlightened        Sat Dec  6 16:43   20/633   This is the subject
O  4 Enlightened        Sat Dec  6 16:44   20/633   This is the subject
U  5 Mail Delivery Syst Sat Dec  6 16:50   74/2425  Undelivered Mail Returned to Sender
U  6 Enlightened        Sat Dec  6 16:51   19/632   This is mutts subject
U  7 Enlightened        Sat Dec  6 16:52   19/647   This is mutts subject
?
```
我们可以使用 ? 来查看帮助，例如查看下一封邮件是n, 退出是q。

日常生活工作中我们可能不会是通过 mail 来收发邮件，但它在我们配置邮件服务器时很有用，我们可以知道 postfix 是否是正常工作的。

如果postfix并没有按我们预期那样正常工作，我们可以通过查看系统日志和邮件日志来定位问题:

```
// system log
$ sudo tail -f /var/log/syslog

// mail log
$ suod tail -f /var/log/mail.log
```

## Dovecot

### 安装

Dovecot 支持 imap 和 pop3，这里我用的 imap,

```
$ aptitude install dovecot-core dovecot-imapd
```

### 配置

Dovecot 的配置文档要比 postfix 好得多，只需要参考它的 Dovecot installation。

* Checking where mail is delivered to

我们的邮件地址是 ~/Maildir, 更新到配置文件中:

```
/etc/dovecot/conf.d/10-mail.conf:
mail_location = maildir:~/Maildir
```

* Configuring Dovecot

```
// Find Dovecot configuration file location using:
$ doveconf -n | head -n1
/etc/dovecot/dovecot.conf

// Authentication
// 由于我们打算使用虚拟用户，所以我们这里创建一个类似密码的文件:
$ echo "$USER:{PLAIN}password:$UID:$GID::$HOME" > users
$ sudo mv users /etc/dovecot/

$GID 这个环境变量可能没值，我们可以手动编辑，首先找出gid：
id $USER

然后把gid 加到对应的字段里面:
$ sudo vim /etc/dovecot/users

// /etc/dovecot/conf.d/10-auth.conf
// Add '#' to comment out the system user login for now:
    #!include auth-system.conf.ext

// Remove '#' to use passwd-file:
!include auth-passwdfile.conf.ext

/etc/dovecot/conf.d/auth-passwdfile.conf.ext
passdb {
      driver = passwd-file
        args = scheme=CRYPT username_format=%u /etc/dovecot/users
}
userdb {
      driver = passwd-file
        args = username_format=%u /etc/dovecot/users
}

// Verify with doveconf -n passdb userdb that the output looks like above (and there are no other passdbs or userdbs).

// Plaintext Authentication
To allow any Authentication without SSL, disable SSL in the conf.d/10-ssl.conf file.
/etc/dovecot/conf.d/10-ssl.conf:
ssl = no

Until SSL is configured, allow plaintext authentication in the conf.d/10-auth.conf file. You probably want to switch this back to "yes" afterward.
/etc/dovecot/conf.d/10-auth.conf:
disable_plaintext_auth = no
```

* Running Dovecot

```
// Start Dovecot
$ sudo service dovecot start

// Stop Dovecot
$sudo service dovecot stop

// Restart 
$sudo service dovecot restart

// Verify
$ sudo ps auxw|grep "dovecot"
```

* Testing that everything works

```
// Check that it's listening
$ telnet localhost 143
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE STARTTLS AUTH=PLAIN] Dovecot ready.

If you got "connection refused", make sure that Dovecot is configured to serve the imap protocol and listening on the expected interfaces/addresses. The simplest way to do that would be using doveconf(1):
$ sudo doveconf protocols listen
protocols = imap pop3 lmtp sieve
listen = *, ::

Next check that it also works from remote host:
$ telnet imap.example.com 143
Trying 1.2.3.4...
Connected to imap.example.com.
Escape character is '^]'.
* OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE STARTTLS AUTH=PLAIN] Dovecot ready.

Check that it's allowing logins
% telnet localhost 143
a login "username" "password"
a OK Logged in.

Check that it's allowing remote logins
$ telnet imap.example.com 143
a login "username" "password"

Check that it finds INBOX
b select inbox
* FLAGS (\Answered \Flagged \Deleted \Seen \Draft)
* OK [PERMANENTFLAGS (\Answered \Flagged \Deleted \Seen \Draft \*)] Flags permitted.
* 1 EXISTS
* 1 RECENT
* OK [UIDVALIDITY 1106186941] UIDs valid
* OK [UIDNEXT 2] Predicted next UID
b OK [READ-WRITE] Select completed.

Make a graceful exit
e logout
* BYE Logging out
e OK Logout completed.

```

* Finishing the test installation

最后我们要开启 ssl, 关闭明文密码验证, [Let's Encrypt](https://letsencrypt.org/)可以提供免费的 SSL 证书:

```
// Enable the Jessie backports repo
For jessie add this line
deb http://ftp.debian.org/debian jessie-backports main
to your source.list(/etc/sources.list)

Run apt-get update

// Temporarily stop apache
$ sudo service apache2 stop

// sudo certbot certonly --standalone -d example.com

All generated keys and issued certificates can be found in /etc/letsencrypt/live/$domain.

/etc/dovecot/conf.d/10-ssl.conf:
ssl = yes 

ssl_cert = </etc/letsencrypt/live/mail.tennshop.com/fullchain.pem
ssl_key = </etc/letsencrypt/live/mail.tenneshop.com/privkey.pem

/etc/dovecot/conf.d/10-auth.conf:
disable_plaintext_auth = yes 

Test using imaps port (assuming you haven't disabled imaps port):
$ openssl s_client -connect imap.example.com:993
* OK Dovecot ready.

Test using imap port and STARTTLS command (works also with imap port):
$ openssl s_client -connect imap.example.com:143 -starttls imap
* OK Dovecot ready.
```

## Mail

现在我们可以在 Mac 上使用 Mail.app 来收信了，但还是不能发信，postfix 默认不支持外域发信，我们需要一种方法获取内域相同的权限。为了解决这个问题， Postfix 支持 SASL.

Configure SASL authentication in the Postfix SMTP server

由于 SASL 的实现和 Postfix 是分开的，所以在 Postfix SMTP 服务器上配置 SASL 授权会有两个不同的内容：

* 配置 SASL 实现来提供一系列合适的机制来进行 SASL 认证，决定好使用哪个 SASL 实现后，配置认证的后端，它通过系统的密码文件或其他数据库来认证远端的 SMTP 终端；
* 配置 Postfix SMTP 服务器使能 SASL 认证，去授权终端中继邮件或控制终端可以使用哪些邮件发送地址;

Which SASL Implementations are supported?

Currently the Postfix SMTP server supports the Cyrus SASL and Dovecot SASL implementations.

To find out what SASL implementations are compiled into Postfix, use the following commands:

```
% postconf -a (SASL support in the SMTP server)
% postconf -A (SASL support in the SMTP+LMTP client)
```

Configuring Dovecot SASL

Dovecot 独立地去认证它的 POP/IMAP 终端，Postfix 使用 Dovecot SASL 时会复用这部分配置。

Postfix 到 Dovecot SASL 的通信有两种途径:UNIX-domain socket or TCP socket。从更好的安全性角度出发，我们选择使用 UNIX-domain socket.

The following fragment for Dovecot version 2 assumes that the Postfix queue is under /var/spool/postfix/.

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

Enabling SASL authentication and authorization in the Postfix SMTP server

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

开启 SASL 认证:

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

Unencrypted SMTP session

```
/etc/postfix/main.cf:
smtpd_sasl_security_options = noanonymous
```

Encrypted SMTP session

```
/etc/postfix/main.cf:
smtpd_sasl_tls_security_options = $smtpd_sasl_security_options
```

To offer SASL authentication only after a TLS-encrypted session has been established specify this:

```
/etc/postfix/main.cf:
smtpd_tls_auth_only = yes
```

After the client has authenticated with SASL, the Postfix SMTP server decides what the remote SMTP client will be authorized for. Examples of possible SMTP clients authorizations are:

* Send a message to a remote recipient.
* Use a specific envelope sender in the MAIL FROM command.

These permissions are not enabled by default.

Mail relay authorization

With permit_sasl_authenticated the Postfix SMTP server can allow SASL-authenticated SMTP clients to send mail to remote destinations. Examples:

```
    # With Postfix 2.10 and later, the mail relay policy is
    # preferably specified under smtpd_relay_restrictions.
/etc/postfix/main.cf:
smtpd_relay_restrictions =
permit_mynetworks
permit_sasl_authenticated
reject_unauth_destination
    # Older configurations combine relay control and spam control under
    # smtpd_recipient_restrictions. To use this example with Postfix ≥
    # 2.10 specify "smtpd_relay_restrictions=".
/etc/postfix/main.cf:
smtpd_recipient_restrictions =
permit_mynetworks
permit_sasl_authenticated
reject_unauth_destination
...other rules...
```

Envelope sender address authorization

By default an SMTP client may specify any envelope sender address in the MAIL FROM command. That is because the Postfix SMTP server only knows the remote SMTP client hostname and IP address, but not the user who controls the remote SMTP client.

This changes the moment an SMTP client uses SASL authentication. Now, the Postfix SMTP server knows who the sender is. Given a table of envelope sender addresses and SASL login names, the Postfix SMTP server can decide if the SASL authenticated client is allowed to use a particular envelope sender address:

```
/etc/postfix/main.cf:
smtpd_sender_login_maps = hash:/etc/postfix/controlled_envelope_senders

smtpd_recipient_restrictions =
...
reject_sender_login_mismatch
permit_sasl_authenticated
...
```

The controlled_envelope_senders table specifies the binding between a sender envelope address and the SASL login names that own that address:

```
/etc/postfix/controlled_envelope_senders
    # envelope sender           owners (SASL login names)
john@example.com            john@example.com
helpdesk@example.com        john@example.com, mary@example.com
postmaster                  admin@example.com
@example.net                barney, fred, john@example.com, mary@example.com
```

With this, the reject_sender_login_mismatch restriction above will reject the sender address in the MAIL FROM command if smtpd_sender_login_maps does not specify the SMTP client's login name as an owner of that address.

Testing SASL authentication in the Postfix SMTP server

To test the server side, connect (for example, with telnet) to the Postfix SMTP server port and you should be able to have a conversation as shown below. 

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

To test this over a connection that is encrypted with TLS, use openssl s_client instead of telnet:

```
% openssl s_client -connect server.example.com:25 -starttls smtp
...
220 server.example.com ESMTP Postfix
EHLO client.example.com
...see above example for more...
```

PLAIN 的授权方式要求用户名和密码一起base64编码提交， LOGIN 的话则是先提交base64编码的用户名，然后是base64编码的密码, base64编码的话可以找个[在线编码工具](https://www.base64encode.org/)：

```
auth login 
334 VXNlcm5hbWU6
c2VydmljZUBoZWVwLmNx
334 UGFzc3dvcmQ6
xxxxxxxx
235 Authentication successful
```

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

我们可以用 SMTP 命令来测试是否能在外域正常发送邮件:

```
>MAIL FROM: sender@example.com

>250 sender <sender@example.com> ok

>RCPT to: recipient@domain.com

>250 recipient <recipient@domain.com> ok

>DATA

>354 go ahead

>Subject: Hi smtp mail

>hello mail

>.

>250 ok:  Message 1763097690 accepted
```

一切都正常工作之后，我们就可以在 Mac 和 iPhone 上配置 Mail 来收发邮件了，由于我们只让 Dovecot 支持了 IMAP,所以收件服务器我们选择 imap。用户名和密码就是我们为 Dovecot 在 /etc/dovecot/users 中配置的用户名密码。邮件地址则是我们为 Postfix 添加的邮件地址，而且这些邮件邮件地址应该在/etc/postfix/controlled_envelop_senders中和SASL 用户做了关联，这个很好理解，就是登录成功的用户他可以用那几个邮件地址发送邮件。

##Reference:
[How To Set Up a Postfix E-Mail Server with Dovecot](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-postfix-e-mail-server-with-dovecot#postfix)  
[How To Install and Configure Postfix on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-configure-postfix-on-ubuntu-16-04#conclusion)  
[Postfix](http://www.postfix.org/)  
[Dovecot](http://wiki.dovecot.org/)  
[Linux mail command examples – send mails from command line](http://www.binarytides.com/linux-mail-command-examples/)
[SMTP的相关命令](http://www.cnblogs.com/cocowool/archive/2012/03/14/2395390.html)
[SMTP Commands and Definitions](https://technet.microsoft.com/en-us/library/aa996114(v=exchg.65).aspx)
