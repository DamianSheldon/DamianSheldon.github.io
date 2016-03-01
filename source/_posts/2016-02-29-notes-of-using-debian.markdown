---
layout: post
title: "Notes Of Using Debian"
date: 2016-02-29 21:46:20 +0800
comments: true
categories: [Archives]
keywords: Debian
discription: Notes Of Using Debian
---
###1. Install pear on Debian
A:

```
$ wget http://pear.php.net/go-pear.phar
$ sudo php go-pear.phar

Below is a suggested file layout for your new PEAR installation.  To
change individual locations, type the number in front of the
directory.  Type 'all' to change all of them or simply press Enter to
accept these locations.

 1. Installation base ($prefix)                   : /home/meiliang/pear
 2. Temporary directory for processing            : /tmp/pear/install
 3. Temporary directory for downloads             : /tmp/pear/install
 4. Binaries directory                            : /home/meiliang/pear/bin
 5. PHP code directory ($php_dir)                 : /home/meiliang/pear/share/pear
 6. Documentation directory                       : /home/meiliang/pear/docs
 7. Data directory                                : /home/meiliang/pear/data
 8. User-modifiable configuration files directory : /home/meiliang/pear/cfg
 9. Public Web Files directory                    : /home/meiliang/pear/www
10. System manual pages directory                 : /home/meiliang/pear/man
11. Tests directory                               : /home/meiliang/pear/tests
12. Name of configuration file                    : /home/meiliang/.pearrc

1-12, 'all' or Enter to continue: 1
(Use $prefix as a shortcut for '/home/meiliang/pear', etc.)
Installation base ($prefix) [/home/meiliang/pear] : /usr/local/bin/pear

$ vim .bashrc
# Append the /usr/local/bin/pear directory in the user's home directory onto the PATH environment variable
export PATH=$PATH:/usr/local/bin/pear/bin

// Reload bash configure file
$ source .bashrc

$ echo $PATH
/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/local/bin/pear/bin

// Check wether it works
$ pear version
PEAR Version: 1.10.1
PHP Version: 5.6.17-0+deb8u1
Zend Engine Version: 2.6.0
Running on: Linux Debian 3.16.0-4-amd64 #1 SMP Debian 3.16.7-ckt11-1 (2015-05-24) x86_64
```

###2. Install PHP GD extension on Debian
A:

```
$ sudo aptitude install php5-gd

// How do I verify that php5-gd support loaded or not?
// Type the following command at a shell prompt:
$ php5 -m | grep -i gd

Sample outputs:

gd
```

Reference:http://www.cyberciti.biz/faq/ubuntu-linux-install-or-add-php-gd-support-to-apache/

###3. Install PHP cURL extension on Debian
A:

```
$ sudo aptitude install php5-curl
```

Reference:http://stackoverflow.com/questions/20073676/how-do-i-install-php-curl-on-linux-debian

###4. Install PHP mCrypt extension on Debian 
A:

```
$ aptitude search mCrypt
p   libmcrypt-dev                                                                    - De-/Encryption Library development files
p   libmcrypt4                                                                       - De-/Encryption Library
p   libtomcrypt-dev                                                                  - static library, header files and documentation for libtomcrypt
p   libtomcrypt0                                                                     - public domain open source cryptographic toolkit
p   mcrypt                                                                           - Replacement for old unix crypt(1)
p   php5-mcrypt                                                                      - MCrypt module for php5

$ sudo aptitude install php5-mcrypt
$ sudo service apache2 restart
```


