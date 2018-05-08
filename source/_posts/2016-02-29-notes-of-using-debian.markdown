---
layout: post
title: "Debian 使用笔记"
date: 2016-02-29 21:46:20 +0800
comments: true
categories: [Archives]
keywords: Debian
description: Notes Of Using Debian
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
<!-- more -->
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

###5. Default installation layouts for Apache HTTPD on Debian
A:

```
ServerRoot              ::      /etc/apache2
DocumentRoot            ::      /var/www
Apache Config Files     ::      /etc/apache2/apache2.conf
                        ::      /etc/apache2/ports.conf
Default VHost Config    ::      /etc/apache2/sites-available/default, /etc/apache2/sites-enabled/000-default
Module Locations        ::      /etc/apache2/mods-available, /etc/apache2/mods-enabled
ErrorLog                ::      /var/log/apache2/error.log
AccessLog               ::      /var/log/apache2/access.log
cgi-bin                 ::      /usr/lib/cgi-bin
binaries (apachectl)    ::      /usr/sbin
start/stop              ::      /etc/init.d/apache2 (start|stop|restart|reload|force-reload|start-htcacheclean|stop-htcacheclean)
```

Notes:

1. The Debian/Ubuntu layout is fully documented in /usr/share/doc/apache2/README.Debian
2. Debian/Ubuntu use symlinks to configure vhosts and load modules. Configuration files are created in their respective sites-available and mods-available directories. To activate vhosts and modules, symlinks are created in the respective sites-enabled and mods-enabled directories to the config files in either sites-available and mods-available. Debian provides scripts to handle this process called 'a2ensite' and 'a2enmod' which activates vhosts and modules.
3. The default vhost is defined in /etc/apache2/sites-available/default, and overrides the DocumentRoot set in the server context.

Reference:http://wiki.apache.org/httpd/DistrosDefaultLayout#Debian.2C_Ubuntu_.28Apache_httpd_2.x.29:

###6. Can't connect to PPTP VPN with ufw enabled on Debian 3.16.7-ckt11-1 with kernel 3.16.0-4-amd64

A: Allow Port 1723 on UFW

```
# ufw allow 1723
```

This is caused by a change for security reason in kernel 3.18 [1]. There are two ways to fix this.

* First approach is adding this rule to the file /etc/ufw/before.rules before the line # drop INVALID packets ... `-A ufw-before-input -p 47 -j ACCEPT`
* Second approach is manually loading the nf_conntrack_pptp module. You can do this by running `$sudo modprobe nf_conntrack_pptp`

To load this module on every boot on Ubuntu, add it to the file /etc/modules.

Reference:http://askubuntu.com/questions/572497/cant-connect-to-pptp-vpn-with-ufw-enabled-on-ubuntu-14-04-with-kernel-3-18

http://silverlinux.blogspot.com/2012/05/how-to-pptp-vpn-on-ubuntu-1204-pptpd.html

##7.OpenVPN server configure correct, but client can't brower web site block by GTW.
A:There is a crue the the log of VPN client issue client DNS be used. So I think this may be the problem. YES, client can brower all web site after remove all client DNS settings.

```
192.168.8.1
202.103.96.112
8.8.8.8
208.67.222.222
208.67.220.220
```

Reference:https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-debian-8
https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-14-04#step-5---installing-the-client-profile

##8.How to stop rpcbind and rpc.statd on debian?
A:Both of these services are only needed if you plan on using NFS for file sharing. They are otherwise unneeded and are a potential security risk.

```bash
$sudo systemctl stop rpcbind.service
$sudo systemctl disable rpcbind.service

$sudo systemctl stop nfs-common.service
$sudo systemctl disable nfs-common.service

```
Reference:[Is it safe to remove rpcbind?](https://www.digitalocean.com/community/questions/is-it-safe-to-remove-rpcbind)  
[rpc.statd running on system not using NFS](https://unix.stackexchange.com/questions/20356/rpc-statd-running-on-system-not-using-nfs?utm_medium=organic&utm_source=google_rich_qa&utm_campaign=google_rich_qa)  

