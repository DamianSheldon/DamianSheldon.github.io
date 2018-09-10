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
3.	直接在php建立连接的时候指定my.socket的位置（官方文档：mysqli_connect）。比如： `$db = new MySQLi('localhost', 'root', 'root', 'my_db', '3306', '/var/run/mysqld/mysqld.sock')`   

Reference:[mysqli不能使用localhost，请问这是怎么回事？](https://segmentfault.com/q/1010000000328531)  

###4.超链接元素的 onclick 方法直接使用表单元素的名字调用其 submit 方法。
A:

```html
<form name="bm_form" action="/delete_bms" method="POST">
	<tr>
		<td><a href="https://www.google.com">https://www.google.com</a></td>
		<td><input type="checkbox" name="del_me" value="https://www.google.com" /></td>
	</tr>
	
	<tr>
		<td><a href="https://www.twitter.com">https://www.twitter.com</a></td>
		<td><input type="checkbox" name="del_me" value="https://www.twitter.com" /></td>
	</tr>
</form>

<a href="#" onclick="bm_form.submit();">Delete BM</a>
	
```

这里展示了可以直接使用表单元素的名字直接来提交的技巧。  

###5.Ionic - Cannot find module “.”
A:Just remove all imports that have /umd at the final.
In my case, I changed: 
`import { IonicPageModule } from 'ionic-angular/umd';`
To: 
`import { IonicPageModule } from 'ionic-angular';`

Reference:[Ionic 2 - Runtime error Cannot find module “.”](https://stackoverflow.com/questions/43236971/ionic-2-runtime-error-cannot-find-module)  

###6.Google maps report google is not define in ionic app.
A:

1. go to your project and do “ionic plugin add cordova-plugin-whitelist”
2. add CSP meta

```
<meta http-equiv="Content-Security-Policy" content="script-src 'self' https://maps.googleapis.com/ https://maps.gstatic.com/ https://mts0.googleapis.com/ 'unsafe-inline' 'unsafe-eval'">
```

Live reload unfortunately seems not work, I get this error:

```
Refused to load the script 'http://localhost:35729/livereload.js?snipver=1' because it violates the following Content Security Policy directive: "script-src 'self' https://maps.googleapis.com/ https://maps.gstatic.com/ https://mts0.googleapis.com/ 'unsafe-inline' 'unsafe-eval'".
```

After some google, update CSP meta works, new CSP meta as follow:

```
  <meta http-equiv="Content-Security-Policy" content="script-src localhost:35729 'self' https://maps.googleapis.com/ https://maps.gstatic.com/ https://mts0.googleapis.com/ 'unsafe-inline' 'unsafe-eval'">
```

Reference:[Ionic + Google Maps: ReferenceError: google is not defined](https://forum.ionicframework.com/t/ionic-google-maps-referenceerror-google-is-not-defined/22550)  
[Solution for livereload problems with new CSP rules](https://forum.ionicframework.com/t/solution-for-livereload-problems-with-new-csp-rules/25449)  

###7.Error: No provider for Navbar!
A:Normally navbar don't provide with injector, we should access like follow:

```
// Template
<ion-navbar #navbar color="primary">
    <ion-title>Whatever</ion-title>
    <ion-buttons right>
      <button icon-only ion-button>
        <ion-icon name='pause'></ion-icon>
      </button>
    </ion-buttons>
</ion-navbar>
```

```
// Typescript
export class Page {

@ViewChild('navbar') navBar: Navbar;

}
```

Reference:[Error in ./HomePage class HomePage - caused by: No provider for Navbar!](https://forum.ionicframework.com/t/error-in-homepage-class-homepage-caused-by-no-provider-for-navbar/82530)  

###8.How to custom back button of ionic app?
A:

```
<ion-header>
  <ion-navbar color='danger' hideBackButton>
    <ion-title>product page</ion-title>
    	<ion-buttons left>
		<button ion-button navPop icon-only>
              <ion-icon ios="ios-arrow-forward" md="md-arrow-forward"></ion-icon>
		</button>
	</ion-buttons>
  </ion-navbar>
</ion-header>
```
Reference:[Change default ion-navbar “back” button ios](https://forum.ionicframework.com/t/change-default-ion-navbar-back-button-ios/47342)  

###9.How to make a div take the remaining height?
A:

1. Absolute Positioning
2. Tables
3. CSS3 calc

Absolute Positioning

```
// html
<div id="inner_fixed">
    I have a fixed height
</div>
 
<div id="inner_remaining">
    I take up the remaining height
</div>
```

{% codeblock css %}

	html, body {
	    height: 100%;
	    width: 100%;
	    margin: 0;
	}
	 
	#inner_fixed {
	    height: 100px;
	    background-color: grey;
	}
	 
	#inner_remaining {
	    background-color: #DDDDDD;    
	 
	    position: absolute;
	    top: 100px;
	    bottom: 0;
	    width: 100%; 
	}

{% endcodeblock %}

pros

* easy to implement
* intuitive

cons

* tedious to maintain (hard-coded positions)

Tables

```
// html
<div id="outer">
    <div id="inner_fixed">
        I have a fixed height
    </div>
 
    <div id="inner_remaining">
        I take up the remaining height
    </div>
</div>
```

{% codeblock css %}

	html, body, #outer {
	    height: 100%;
	    width: 100%;
	    margin: 0;
	}
	 
	#outer {
	    display: table;
	}
	 
	#inner_fixed {
	    height: 100px;
	    background-color: grey;
	 
	    display: table-row;
	}
	 
	#inner_remaining {
	    background-color: #DDDDDD;
	 
	    display: table-row;    
	}
{% endcodeblock %}

pros

* rather “clean” solution
* no hard-coded values, other elements can change their height

cons

* might cause some side-effects with the layout

CSS3 calc

```
// html
<div id="inner_fixed">
    I have a fixed height
</div>
 
<div id="inner_remaining">
    I take up the remaining height
</div>
```

{% codeblock css %}

	html, body {
	    height: 100%;
	    width: 100%;
	    margin: 0;
	}
	 
	#inner_fixed {
	    height: 100px;
	    background-color: grey;
	}
	 
	#inner_remaining {
	    background-color: #DDDDDD;
	 
	    height: calc(100% - 100px);    
	}
{% endcodeblock %}

pros

* easy to implement
* less code than the other solutions

cons

* the calc function is rather new (no support for older browsers)
* tedious to maintain (hard-coded height)

Reference:[How to make a div take the remaining height](https://www.whitebyte.info/programming/css/how-to-make-a-div-take-the-remaining-height)  

###10.Property 'of' does not exist on type 'typeof Observable
A:
for Angular >= 6.0.0
uses RxJS 6.0.0 

```

import { of } from 'rxjs';

```

And its usage has been changed, you no longer call it off of Observable:

```

of('token');

```

for Angular <= 5.x.xx

```

import 'rxjs/add/observable/of';

Observable.of('token');

```

Reference:[Property 'of' does not exist on type 'typeof Observable](https://stackoverflow.com/questions/38067580/property-of-does-not-exist-on-type-typeof-observable)  



