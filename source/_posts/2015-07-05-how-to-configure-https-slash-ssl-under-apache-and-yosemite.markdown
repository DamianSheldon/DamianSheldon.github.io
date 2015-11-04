---
layout: post
title: "如何在 Mac 上开启 Apache, PHP, MySQL"
date: 2015-07-05 21:24:55 +0800
comments: true
categories: [Archives]
keywords: https, SSL, Mac, Apache, MySQL
discription: 
---
## Apache
### 开启 Apache 
1. Applications > Utilities > Terminal > `sudo apachectl start`
2. Open http://localhost with safari you can see "It's works!".

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


### 添加VirtualHost，DocumentRoot为/Users/username/Sites/,Error message “Forbidden You don't have permission to access / on this server”

Solution:
I faced the same issue, but I solved it by setting the options directive either in the global directory setting in the httpd.conf or in the specific directory block in httpd-vhosts.conf:

Options Indexes FollowSymLinks Includes ExecCGI
By default, your global directory settings is (httpd.conf line ~188):

<Directory />
     AllowOverride none
     Require all granted
</Directory>
set the options to : Options Indexes FollowSymLinks Includes ExecCGI

Finally, it should look like:

<Directory />
     Options Indexes FollowSymLinks Includes ExecCGI
     AllowOverride All
     Require all granted
</Directory>
Also try changing "Order deny,allow" and "Allow from all" lines by "Require all granted"

Reference:http://stackoverflow.com/questions/10873295/error-message-forbidden-you-dont-have-permission-to-access-on-this-server

Reference:http://webdevstudios.com/2013/05/24/how-to-set-up-ssl-with-osx-mountain-lions-built-in-apache/   
http://www.akadia.com/services/ssh_test_certificate.html  
http://charles.lescampeurs.org/2014/04/01/how-to-configure-httpsssl-under-apache-and-osx  
http://coolestguidesontheplanet.com/get-apache-mysql-php-and-phpmyadmin-working-on-osx-10-11-el-capitan/

