---
layout: post
title: "如何在 Mac 上开启 Apache, PHP, MySQL"
date: 2015-07-05 21:24:55 +0800
comments: true
categories: [Archives, Web Development]
keywords: https, SSL, Mac, Apache, MySQL
discription:
---
## Apache
### 开启 Apache
1. Applications > Utilities > Terminal > `sudo apachectl start`
2. Open http://localhost with safari you can see "It's works!".

### 开启为系统的守护进程

```
$ launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist
```

### 配置 HTTPS 的详细步骤：  

1. 在配置文件中使能HTTPS/SSL：/private/etc/apache2/httpd.conf

	* 去掉 SSL 模块前的注释使能它:  
	```
	LoadModule ssl_module libexec/apache2/mod_ssl.so
	```
   * 去掉包含 SSL 配置文件前的注释，以便顶层配置文件能包含它:  
   ```
   Include /private/etc/apache2/extra/httpd-ssl.conf
   ```

2. 定制/private/apache2/extra/httpd-ssl.conf  

	* 更新 DocumentRoot 指向你的 Web 根目录:  
	```
	DocumentRoot "/Users/dongmeiliang/Sites"
	```

	* 修改 ServerName 成类似这样:  
	```
	ServerName localhost:443
	```
	* 指定 SSLCertificateFile  
	```
	SSLCertificateFile "/private/etc/apache2/ssl/ssl.crt"
	```

	* 指定 SSLCertificateKeyFile   
	```
	SSLCertificateKeyFile "/private/etc/apache2/ssl/ssl.key"
	```
	* 在 SSLCACertificatePath 和 SSLCARevocationPath 配置项前加上 # ，注释掉它们。
<!-- more -->
3. 制作自签名的证书

	1) Generate a Private Key  
	```
	openssl genrsa -des3 -out server.key 1024
	```

	2) Generate a CSR (Certificate Signing Request)  
	```
	openssl req -new -key server.key -out server.csr
	```

	3) Remove Passphrase from Key  
	```
	cp server.key server.key.org  
	```  
	```
	openssl rsa -in server.key.org -out server.key
	```

	4) Generating a Self-Signed Certificate  
	```
	openssl x509 -req -days 365 -in server.csr -signkey server.key -out 	server.crt
	```

4. 拷贝证书到配置文件指定的目录下  
```
sudo cp server.crt /private/etc/apache2/ssl/ssl.crt
sudo cp server.key /private/etc/apache2/ssl/ssl.key
```


5. 重启Apache，打开 https://localhost 测试是否工作正常  

问题：Invalid command 'SSLMutex', perhaps misspelled or defined by a module not included in the server configuration  
解决办法：注释掉/private/etc/apache2/extra/httpd-ssl.conf中SSLMutex解决了这个问题。

## PHP
1. Open /etc/apache2/httpd.conf with your favorite editor (eg: Vim);
2. Uncomment LoadModule php5_module libexec/apache2/libphp5.so
3. `sudo apachectl restart`
4. Test with phpinfo.php, its content is "<?php phpinfo(); ?>"

## MySQL
1. [Download from the MySQL site](http://dev.mysql.com/downloads/mysql/), click No thanks, just take me to the downloads!;
2. Choose Mac OS X ver. 10.9 (x86, 64-bit), DMG Archive, because there is an issue with this version and El Capitan in that it won’t start on reboot – it will need to be started via command line explained below.
3. Starting MySQL
	* Via Preference Pane
	* Command line
		```
		sudo /usr/local/mysql/support-files/mysql.server start
		```
4. Add MySQL directory to shell path
	* Open ~/.bash_profile
	* export PATH="/usr/local/mysql/bin:$PATH"
	* source ~/.bash_profile

5. Change root password
	```
	mysql -u root -p
	mysql> SET PASSWORD FOR
    -> 'root'@'localhost' = PASSWORD('mypass');
	```

6. Fix the 2002 MySQL Socket error
	Fix the looming 2002 socket error – which is linking where MySQL places the socket and where OSX thinks it should be, MySQL puts it in /tmp and OSX looks for it in /var/mysql the socket is a type of file that allows mysql client/server communication.
	* sudo mkdir /var/mysql
	* sudo ln -s /tmp/mysql.sock /var/mysql/mysql.sock

7. How to Start & Stop MySQL Manually in OS X

	```
	// Start MySQL
	$ sudo /usr/local/mysql/support-files/mysql.server start

	// Stop MySQL
	$ sudo /usr/local/mysql/support-files/mysql.server stop

	// Restart MySQL
	$ sudo /usr/local/mysql/support-files/mysql.server restart
	```

	8. Where is MySQL's error log
	Which by default is the host_name.err file in the data directory, in my situation(OS X El Capitan), it locate in /usr/local/mysql/data/DongMeiliangsMacBook-Pro.local.err.

	9. View MySQL's status

	```
	$ sudo /usr/local/mysql/support-files/mysql.server status
	```

  10. ERROR 2002 (HY000): Can ' t connect to local MySQL server through socket '/tmp/mysql.sock'

	```
	// View MySQL status
	$ /usr/local/mysql/support-files/mysql.server status
	/usr/local/mysql/support-files/mysql.server: line 365: pidof: command not found
 	ERROR! MySQL is not running

	// View MySQL's error log
	$ sudo vim /usr/local/mysql/data/DongMeiliangsMacBook-Pro.local.err

	// Find today's log
	2015-12-12T11:21:14.012289Z 0 [ERROR] InnoDB: Unable to lock ./ibdata1 error: 35
	2015-12-12T11:21:14.012335Z 0 [Note] InnoDB: Check that you do not already have another mysqld process using the same InnoDB data or log files.

	// Is there mysql process
	$ sudo pstress | grep 'mysql'

	|-+= 30102 root /bin/sh /usr/local/mysql/bin/mysqld_safe --user=mysql
	| \--- 30203 _mysql /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/us
	|--= 30207 _mysql /usr/local/mysql/bin/mysqld --user=_mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --log-error=/usr

	// Unfortunate, there are indeed two mysql procee, so kill one
	$ sudo kill -9  30102

	// Check we succeed kill a mysql process
	$ sudo pstree | grep 'mysql'

	// Unfortunate, when I killed it, it will auto start, so I have to disable auto start it.
	|-+= 30210 root /bin/sh /usr/local/mysql/bin/mysqld_safe --user=mysql
	| \--- 30203 _mysql /usr/local/mysql/bin/mysqld --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --user=mysql --log-error=/us
	|--= 30207 _mysql /usr/local/mysql/bin/mysqld --user=_mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --log-error=/usr

	// Figure out who auto start mysql
	$ sudo launchctl list | grep 'mysql'
	30102	0	com.mysql.mysqld
	30374	1	com.oracle.oss.mysql.mysqld

	// Unloads mysql auto start service
	$ sudo launchctl unload -w /Library/LaunchDaemons/com.mysql.mysql.plist
	$ sudo launchctl unload -w /Library/LaunchDaemons/com.oracle.oss.mysql.mysqld.plist

	// Stop mysql service
	$ sudo launchctl stop com.mysql.mysqld
	$ sudo launchctl stop com.oracle.oss.mysql.mysqld

	// Start mysql
	$ sudo /usr/local/mysql/support-files/mysql.server start
	Starting MySQL
	. SUCCESS!
	
	```

#### 添加VirtualHost，DocumentRoot为/Users/username/Sites/,Error message “Forbidden You don't have permission to access / on this server”

Solution:
I faced the same issue, but I solved it by setting the options directive either in the global directory setting in the httpd.conf or in the specific directory block in httpd-vhosts.conf:

```
Options Indexes FollowSymLinks Includes ExecCGI

```

By default, your global directory settings is (httpd.conf line ~188):

```
<Directory />
     AllowOverride none
     Require all granted
</Directory>

```

set the options to : `Options Indexes FollowSymLinks Includes ExecCGI`

Finally, it should look like:

```
<Directory />
     Options Indexes FollowSymLinks Includes ExecCGI
     AllowOverride All
     Require all granted
</Directory>
```

Also try changing "Order deny,allow" and "Allow from all" lines by "Require all granted"

Reference:[Error message “Forbidden You don't have permission to access / on this server”](http://stackoverflow.com/questions/10873295/error-message-forbidden-you-dont-have-permission-to-access-on-this-server)

Reference:
[How to Set Up SSL with OSX Mountain Lion’s Built-In Apache](http://webdevstudios.com/2013/05/24/how-to-set-up-ssl-with-osx-mountain-lions-built-in-apache/)   
[How to create a self-signed SSL Certificate](http://www.akadia.com/services/ssh_test_certificate.html)  
[How to configure HTTPS/SSL under Apache and OSX](http://charles.lescampeurs.org/2014/04/01/how-to-configure-httpsssl-under-apache-and-osx)  
[Get Apache, MySQL, PHP and phpMyAdmin working on OSX 10.11 El Capitan](http://coolestguidesontheplanet.com/get-apache-mysql-php-and-phpmyadmin-working-on-osx-10-11-el-capitan/)


