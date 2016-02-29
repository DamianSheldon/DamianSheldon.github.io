---
layout: post
title: "Web Development Notes Part One"
date: 2015-12-12 22:27:22 +0800
comments: true
categories: [Archives, Web Development]
keywords: HTML, CSS, JavaScript, Bootstrap, ionic, GMP, AngularJS, JQuery
discription: Web Development Notes Part One
---

###1. What is href=“#” and why is it used?

Solution:A hashtag - # within a hyperlink specifies an html element id to which the window should be scrolled.  
href="#some-id" would scroll to an element on the current page such as <div id="some-id">.

href="//site.com/#some-id" would go to site.com and scroll to the id on that page.  

href="#" doesn't specify an id name, but does have a corresponding location - the top of the page. Clicking an anchor with href="#" will move the scroll position to the top.

Reference:http://stackoverflow.com/questions/4855168/what-is-href-and-why-is-it-used

###2. How to debug your web page, technology stack is LAMP.  

Solution:View log, figure out what cause problems.

```
$ tail -f /var/log/apache2/error_log
[Fri Dec 11 21:34:57.003657 2015] [:error] [pid 17465] [client ::1:50505] PHP Parse error:  parse error, expecting `','' or `';'' in /Users/dongmeiliang/Sites/knowledgeispower/includes/header.html on line 7
[Fri Dec 11 21:34:57.003701 2015] [:error] [pid 17465] [client ::1:50505] PHP Stack trace:
[Fri Dec 11 21:34:57.003723 2015] [:error] [pid 17465] [client ::1:50505] PHP   1. {main}() /Users/dongmeiliang/Sites/knowledgeispower/index.php:0
```
<!-- more -->
###3. Install mcrypt_filter extension on El capitan

```
// Please make sure the PHP extensions listed below are installed.
mCrypt

// PECL is a repository of PHP extensions that are made available to you via the » PEAR packaging system. 

// There are several options for downloading PECL extensions, such as:
o The pecl install extname command downloads the extensions code automatically, so in this case there is no need for a separate download.
o » http://pecl.php.net/ The PECL web site contains information about the different extensions that are offered by the PHP Development Team. The information available here includes: ChangeLog, release notes, requirements and other similar details.
o pecl download extname PECL extensions that have releases listed on the PECL web site are available for download and installation using the » pecl command. Specific revisions may also be specified.
o SVN Most PECL extensions also reside in SVN. A web-based view may be seen at » http://svn.php.net/viewvc/pecl/. To download straight from SVN, the following sequence of commands may be used:

$ pecl search mCrypt
Retrieving data...0%
Matched packages, channel pecl.php.net:
=======================================
Package       Stable/(Latest) Local
mcrypt_filter 0.1.0 (beta)          Applies mcrypt symmetric encryption using stream filters

$ pecl install mcrypt_filter
Failed to download pecl/mcrypt_filter within preferred state "stable", latest release is version 0.1.0, stability "beta", use "channel://pecl.php.net/mcrypt_filter-0.1.0" to install
install failed

$ pecl install channel://pecl.php.net/mcrypt_filter-0.1.0/mcrypt_filter
Attempting to discover channel "pecl.php.net/mcrypt_filter-0.1.0"...
Attempting fallback to https instead of http on channel "pecl.php.net/mcrypt_filter-0.1.0"...
unknown channel "pecl.php.net/mcrypt_filter-0.1.0" in "channel://pecl.php.net/mcrypt_filter-0.1.0/mcrypt_filter"
invalid package name/package file "channel://pecl.php.net/mcrypt_filter-0.1.0/mcrypt_filter"
install failed

$ pecl install channel://pecl.php.net/mcrypt_filter-0.1.0
Cannot install, php_dir for channel "pecl.php.net" is not writeable by the current user
DongMeiliangsMacBook-Pro:upload dongmeiliang$ sudo pecl install channel://pecl.php.net/mcrypt_filter-0.1.0
Password:
downloading mcrypt_filter-0.1.0.tgz ...
Starting to download mcrypt_filter-0.1.0.tgz (4,586 bytes)
...
checking for mcrypt support... yes, shared
configure: error: mcrypt.h not found. Please reinstall libmcrypt.
ERROR: `/private/tmp/pear/install/mcrypt_filter/configure --with-php-config=/usr/bin/php-config' failed

// Resolve configure: error: mcrypt.h not found. Please reinstall libmcrypt.
Get libmcrypt 2.5.8 from Sourceforge: http://sourceforge.net/projects/mcrypt/files/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz/download

$ tar -zxvf libmcrypt-2.5.8.tar.gz
$ mkdir -p /usr/local/src
$ mv libmcrypt-2.5.8 /usr/local/src/
$ cd /usr/local/src/libmcrypt-2.5.8/
$ ./configure --prefix=/usr/local
$ make && make install

// Continue install mCrypt extension
$ sudo pecl install channel://pecl.php.net/mcrypt_filter-0.1.0
...

12472464 40 -rwxr-xr-x  1 root  wheel  16704 Feb 15 21:41 /private/tmp/pear/install/pear-build-rootEyJ4Jg/install-mcrypt_filter-0.1.0/usr/lib/php/extensions/no-debug-non-zts-20121212/mcrypt_filter.so

Build process completed successfully
Installing '/usr/lib/php/extensions/no-debug-non-zts-20121212/mcrypt_filter.so'
ERROR: failed to write /usr/lib/php/extensions/no-debug-non-zts-20121212/mcrypt_filter.so (copy(/usr/lib/php/extensions/no-debug-non-zts-20121212/mcrypt_filter.so): failed to open stream: Operation not permitted)

We need to disable SIP.

OSX 10.11 El Capitan ships with a new security process referred to as SIP or Security Integrity Protection also known as rootless.

What this essentially does is protect certain system level directories from being changes, notably

o /System
o /sbin
o /usr
o not /usr/local

// Disabling SIP in OSX
o Reboot your Mac into Recovery Mode by rebooting and holding down ‘command’ + ‘r’
o Launch a Terminal session and enter csrutil disable
o Reboot

// Reinstall mCrypt extension
$ sudo pecl install channel://pecl.php.net/mcrypt_filter-0.1.0

...

Build process completed successfully
Installing '/usr/lib/php/extensions/no-debug-non-zts-20121212/mcrypt_filter.so'
install ok: channel://pecl.php.net/mcrypt_filter-0.1.0
configuration option "php_ini" is not set to php.ini location
You should add "extension=mcrypt_filter.so" to php.ini

```
Reference:http://www.nginx.cn/2196.html
http://coolestguidesontheplanet.com/install-mcrypt-for-php-on-mac-osx-10-10-yosemite-for-a-development-server/

###4.Install mcrypt for php on Mac OSX 10.11 El Capitan
o Disable SIP/rootless
You need to disable the new System Integrity Protection to install into some system protected directories – this involves a recovery reboot and disabling with.

```
$ csrutil disable
```

o Getting it on in OS X El Capitan 10.11

```
$ cd ~ ; mkdir mcrypt ; cd mcrypt
```

[Get libmcrypt 2.5.8 from Sourceforge](http://sourceforge.net/projects/mcrypt/files/Libmcrypt/2.5.8/libmcrypt-2.5.8.tar.gz/download),  this is direct download link.

[Get the php code in a tar.gz or .bz2 format](http://php.net/releases/index.php)- (version 5.5.27 is the one that currently ships with OSX 10.11)

Move both of these files that you downloaded into your working directory – mcrypt in this instance, and go back to Terminal

```
$ cd ~/mcrypt
```

Expand both files via the command line or just double click them in the Finder:

```
$ tar -zxvf libmcrypt-2.5.8.tar.gz
$ tar -zxvf php-5.5.27.tar.gz
```

o Configuring libmcrypt

```
$ cd libmcrypt-2.5.8
$ ./configure
$ make
$ sudo make install
```

o Compile mcrypt php Extension

```
$ cd ../php-5.5.27/ext/mcrypt/
$ /usr/bin/phpize
$ ./configure
$ make
$ sudo make install
```

o Enabling mcrypt.so  php Extension

```
// Open /etc/php.ini and add the line below at the end
extension=mcrypt.so

// If there is no php.ini file,  then you need to make one from php.ini.default in the same location like so:
$ sudo cp /etc/php.ini.default /etc/php.ini

// And allow write capability
$ sudo chmod u+w  /etc/php.ini

// Then add the line as above in your favourite text editor:
$ sudo vi /etc/php.ini

// make sure dynamic extensions are set to on…
extension_dl = On

// If it didn’t load you may need to declare the extensions directory in /etc/php.ini
extension_dir = "/usr/lib/php/extensions/no-debug-non-zts-20100525/"
```

o Restart Apache

```
$ sudo apachectl restart
```

That’s it, create a php page with the function phpinfo();  to see if it loaded correctly.

Reference:http://coolestguidesontheplanet.com/install-mcrypt-for-php-on-mac-osx-10-10-yosemite-for-a-development-server/
http://coolestguidesontheplanet.com/what-is-sip-in-osx-10-11-el-capitan/

###4. How to set up Apache MySQL PHP on Debian?
A:https://wiki.debian.org/LaMp

###5. Where is apache error log path on Debian?
A:/var/log/apache2/error.log



