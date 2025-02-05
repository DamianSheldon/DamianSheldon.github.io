---
layout: post
title: "Set up PHP development environment"
date: 2015-09-19 22:25:19 +0800
comments: true
categories: [Archives, Web Development]
keywords: PHP, development, environment
description: How to set up php development environment?
---

###Notes of Set up PHP development environment

1. Install [Eclipse PDT](https://eclipse.org/pdt/#download);  
2. [Install PEAR and PECL on Mac OS X](http://jason.pureconcepts.net/2012/10/install-pear-pecl-mac-os-x/), php.ini locate in /etc on Yosemite;  
3. [Install Xdebug through PEAR/PECL](http://www.xdebug.org/docs/install);  
	Addition configure for Yosemite   
	* `sudo cp /etc/php.ini.default /etc/php.ini`  
	* add "zend_extension=/usr/lib/php/extensions/no-debug-non-zts-20121212/xdebug.so" to php.ini

4. Debugging Local PHP Files (PHP CLI Application)

	* Create a PHP project;
	* Create a PHP file;
	* Set a breakpoint in PHP file;
	* Save file;
	* Create a new debug configuration;
		* Eclipse > Preference... > PHP Executables > Add (Executable path: `which php`)
		* Run > Debug Configurations > Select PHP CLI Application > New   

5. Debugging PHP Web Applications (PHP Web Application)
	* Create a PHP project;
	* Create a PHP file;
	* Create a new debug configuration;
		* Run > Debug Configurations > Select PHP Web Application > New

6. Test with PHPUnit
	* Run > External Tools > External Tools Configurations... > Program > New
	[Main]
	Location:/usr/local/bin/phpunit
	Working Directory:${workspace_loc:/Backend/tests}
	Apply > Run
	[Reference](http://stackoverflow.com/questions/966673/eclipse-pdt-phpunit)

7. Integrate MakeGood (failure)
	* Installing Prerequisite Software
		* PHP 5.5.27 (cli)
		* xdebug-2.3.2
		* eclipse mars(4.5.0)
		* PDT(3.5.0)
		* PHPUnit (4.8.6)
	* Installing MakeGood
		Help -> Install New Software... -> Add... -> http://eclipse.piece-framework.com/

	* Selecting a Testing Framework  

	* Resolving the Dependencies to the Testing Framework
			* Defining User Libraries
				Eclipse -> Preferences... > PHP -> PHP Libraries > New... > OK
				Add External folder... > [Verifying the include path](http://pear.php.net/manual/en/installation.checking.php)
			* Adding User Libraries to the Project

		 fail with PHPUnit_Framework_TestCase class is not available. Fix..., I doubt that MakeGood doesn't support eclipse mars.

	*Testing Framework Specific Configuration  
		PHPUnit
		* Open the properties for the project.
		* Select MakeGood.
		* Select PHPUnit.
		* Specify the XML Configuration File. [eg](http://beagile.biz/a-simple-phpunit-xml-configuration-example/)

	[Reference](http://piece-framework.com/projects/makegood/wiki/MakeGood_User_Guide_1_7_0)

###Throw java.lang.NullPointerException when create a new debug configuration

Solution: Add a PHP Executable, Eclipse > Preference... > PHP Executables > Add (Executable path: `which php`)

###Wait for XDebug session

Solution:Debugging PHP Web Applications时卡在这里，原因是XDebug remote debug没配置好，参考了[Remote Debugging](http://www.xdebug.org/docs/remote)和[Debugging using XDebug](https://wiki.eclipse.org/Debugging_using_XDebug)，发现phpinfo()输出的内容并没有XDebug。于是来开始配置php.ini,参考的[How to set default php.ini to be used, OSX Yosemite](http://stackoverflow.com/questions/27861720/how-to-set-default-php-ini-to-be-used-osx-yosemite)  

```
sudo cp /etc/php.ini.default /etc/php.ini

// Add follow content to php.ini
[XDebug]
zend_extension=/usr/lib/php/extensions/no-debug-non-zts-20121212/xdebug.so
xdebug.remote_enable=On
xdebug.remote_handler=dbgp
xdebug.remote_host=helloworld.local
xdebug.remote_port=9000
;xdebug.remote_connect_back=1
xdebug.remote_mode="req"
xdebug.remote_log="/var/log/xdebug.log"

sudo apachectl restart
```
###/private/tmp/pear/install/xdebug/xdebug.c:25:10: fatal error: 'php.h' file not found

Solution:

```
$ xcode-select --install
```
Reference:http://stackoverflow.com/questions/19531262/cant-phpize-or-configure-an-extension-in-os-x-10-9-mavericks
