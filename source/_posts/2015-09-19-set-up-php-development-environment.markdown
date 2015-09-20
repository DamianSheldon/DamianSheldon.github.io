---
layout: post
title: "Set up PHP development environment"
date: 2015-09-19 22:25:19 +0800
comments: true
categories: [Archives]
keywords: 
discription: 
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
	
	
###Throw java.lang.NullPointerException when create a new debug configuration

Solution: Add a PHP Executable, Eclipse > Preference... > PHP Executables > Add (Executable path: `which php`)