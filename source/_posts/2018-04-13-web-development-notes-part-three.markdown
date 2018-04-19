---
layout: post
title: "Web 开发问题汇总(三)"
date: 2018-04-13 11:07:25 +0800
comments: true
categories: [Archives, Web Development]
keywords: web, jquery, servlet, eclipse
description: Noting problems encounter during web development, every fifteen problem produce a blog, this is the three.
---

###1.How to send email in java?
A:

```java
import java.util.*;
import javax.mail.*;
import javax.mail.internet.*;
import javax.activation.*;

public class SendEmail {

   public static void main(String [] args) {    
      // Recipient's email ID needs to be mentioned.
      String to = "abcd@gmail.com";

      // Sender's email ID needs to be mentioned
      String from = "web@gmail.com";

      // Assuming you are sending email from localhost
      String host = "localhost";

      // Get system properties
      Properties properties = System.getProperties();

      // Setup mail server
      properties.setProperty("mail.smtp.host", host);

      // Get the default Session object.
      Session session = Session.getDefaultInstance(properties);

      try {
         // Create a default MimeMessage object.
         MimeMessage message = new MimeMessage(session);

         // Set From: header field of the header.
         message.setFrom(new InternetAddress(from));

         // Set To: header field of the header.
         message.addRecipient(Message.RecipientType.TO, new InternetAddress(to));

         // Set Subject: header field
         message.setSubject("This is the Subject Line!");

         // Now set the actual message
         message.setText("This is actual message");

         // Send message
         Transport.send(message);
         System.out.println("Sent message successfully....");
      } catch (MessagingException mex) {
         mex.printStackTrace();
      }
   }
}
```

Reference:[Java - Sending Email](http://www.tutorialspoint.com/java/java_sending_email.htm)  
[Send email using java](https://stackoverflow.com/questions/3649014/send-email-using-java)  
<!--more-->

###2.Eclipse error: 'Failed to create the Java Virtual Machine'
A:安装了 Eclipse 提示的升级后再重新打开时，出现上面的报错。Google 一圈后显示可能是 eclipse.ini 的锅，正好我安装了 cpp, java 和 jee 三个版本，尝试打开 java 版本的，正常打开，于是对比它们的 eclipse.ini 文件:

```
$ cp ~/eclipse/jee-oxygen/Eclipse.app/Contents/Eclipse/eclipse.ini  ~/eclipse-workspace/jee-eclipse.ini
cp ~/eclipse/java-oxygen/Eclipse.app/Contents/Eclipse/eclipse.ini ~/eclipse-workspace/java-eclipse.ini

$ diff eclipse-workspace/jee-eclipse.ini eclipse-workspace/java-eclipse.ini
3,4d2
< --launcher.library
< /Users/dongmeiliang/.p2/pool/plugins/org.eclipse.equinox.launcher.cocoa.macosx.x86_64_1.1.551.v20171108-1834
6a5,6
> --launcher.library
> /Users/dongmeiliang/.p2/pool/plugins/org.eclipse.equinox.launcher.cocoa.macosx.x86_64_1.1.551.v20171108-1834
8c8
< org.eclipse.epp.package.jee.product
---
> org.eclipse.epp.package.java.product
18,23d17
< -javaagent:/Users/dongmeiliang/.p2/pool/plugins/com.zeroturnaround.eclipse.optimizer.plugin_1.0.11/agent/eclipse-optimizer-agent.jar
< -server
< -XX:PermSize=256m
< -XX:MaxPermSize=256m
< -XX:+UseParallelGC
< -Xverify:none
32c26
< -Xms512m
---
> -Xms256m
```

总共有五处不同，细看之后发现只有两处可能会产生影响，先尝试将 `-Xms512m` 改成 `-Xms256m`，问题仍然存在，将文件恢复原样，再尝试将 

```
< -javaagent:/Users/dongmeiliang/.p2/pool/plugins/com.zeroturnaround.eclipse.optimizer.plugin_1.0.11/agent/eclipse-optimizer-agent.jar
< -server
< -XX:PermSize=256m
< -XX:MaxPermSize=256m
< -XX:+UseParallelGC
< -Xverify:none
```

删除，问题顺利解决。  

Reference:[Eclipse error: 'Failed to create the Java Virtual Machine'](https://stackoverflow.com/questions/7302604/eclipse-error-failed-to-create-the-java-virtual-machine)  

###3.`Warning: mysqli::__construct(): (HY000/2002): No such file or directory in /code/index.php on line 8 Connection failed: No such file or directory`
A:问题出现的原因：
当主机填写为localhost时MySQL会采用 unix domain socket连接，当主机填写为127.0.0.1时MySQL会采用TCP/IP的方式连接。使用Unix socket的连接比TCP/IP的连接更加快速与安全。这是MySQL连接的特性，可以参考官方文档的说明[4.2.2. Connecting to the MySQL Server](https://dev.mysql.com/doc/refman/5.6/en/connecting.html#id471316):  

> On Unix, MySQL programs treat the host name localhost specially, in a way that is 
> likely different from what you expect compared to other network-based programs. 
> For connections to localhost, MySQL programs attempt to connect to the local server 
> by using a Unix socket file. This occurs even if a --port or -P option is given to 
> specify a port number. To ensure that the client makes a TCP/IP connection to the 
> local server, use --host or -h to specify a host name value of 127.0.0.1, or the IP 
> address or name of the local server. You can also specify the connection protocol 
> explicitly, even for localhost, by using the --protocol=TCP option.

这个问题有以下几种解决方法：

1.	使用TCP/IP代替Unix socket。即在连接的时候将localhost换成127.0.0.1。
2.	修改MySQL的配置文件my.cnf，指定mysql.socket的位置： /var/lib/mysql/mysql.sock (你的mysql.socket路径)。 
3.	直接在php建立连接的时候指定my.socket的位置（官方文档：mysqli_connect）。比如： $db = new MySQLi('localhost', 'root', 'root', 'my_db', '3306', '/var/run/mysqld/mysqld.sock') 
Reference:[mysqli不能使用localhost，请问这是怎么回事？](https://segmentfault.com/q/1010000000328531)  



