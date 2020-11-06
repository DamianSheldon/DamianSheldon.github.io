---
layout: post
title: "Linux 使用笔记(三)"
date: 2020-09-02 11:40:17 +0800
comments: true
categories: [Archives]
keywords: Backup, Restore, CentOS 8, Linux
description: Linux usage notes
---

### 1.系统迁移和备份
A:为了数据安全，我们需要对系统备份，换新电脑或者更换云服务时我们需要迁移系统。Linux 系统备份和迁移的方法很多，我这里打算使用 tar 。  

#### 系统备份 
首先是根据自己的实际情况列出需要备份的目录，通常有：  

* `/etc/` 
* `/home/`
* `/var/spool/mail/`
* `/var/spool/cron/`
* `/root`
* `/usr/local/bin`

然后使用 tar 命令打包：

```
tar -jcv -f /backups/backup-system-20200902.tar.bz2 \
> --exclude=/root/*.bz2 --exclude=/root/*.gz --exclude=/home/loop* \
> /etc /home /var/spool/mail /var/spool/cron /root /usr/local/bin
```


#### 系统恢复
首先可以将备份解压到 `/tmp` 目录，之后使用 rsync 命令复制到对应目录便可恢复。

```
tar -jxv -f /backups/backup-system-20200902.tar.bz2 -C /tmp
rsync -avuz /tmp/etc/ /etc
rsync -avuz /tmp/home/ /home
rsync -avuz /tmp/var/spool/mail/ /var/spool/mail
rsync -avuz /tmp/var/spool/cron/ /var/spool/cron
rsync -avuz /tmp/root/ /root
rsync -avuz /tmp/usr/local/bin/ /usr/local/bin
```
<!--more-->
#### ~~系统迁移
换新电脑或者更换云服务时我们可能不想要上面那么麻烦，而可能想直接迁移系统，至少我是这么想的，这时我们可以使用下面的方法~~：

```
# 全系统备份
tar --create --absolute-names --preserve-permissions --bzip2  --file=/media/sf_Windows10-shared-folder/virtual-box-centos-8.tar.bz2 --exclude=/dev --exclude=/media --exclude=/metainfo --exclude=/mnt --exclude=/proc --exclude=/run --exclude=/sys  --exclude=/tmp --exclude=/var --exclude=/@System.solv /

# 如有需要也可检查备份的文件
tar -tjpPvf /media/sf_Windows10-shared-folder/virtual-box-centos-8.tar.bz2 | less

# 将备份包放到 /tmp 下解压
tar -xjvf virtual-box-centos-8.tar.bz2
```

Reference:  

* [Migrate installation to new hardware](https://wiki.archlinux.org/index.php/migrate_installation_to_new_hardware)  
* [How To Migrate to a New Linux Server](https://www.digitalocean.com/community/tutorial_series/how-to-migrate-to-a-new-linux-server)  

### 2.dnf list plugins command missing
A:The information is provided for nearly all command with "-v" option. See:

Loaded plugins: builddep, changelog, config-manager, copr, debug, debuginfo-install, download, generate_completion_cache, needs-restarting, playground, product-id, repoclosure, repodiff, repograph, repomanage, reposync, subscription-manager, uploadprofile
Updating Subscription Management repositories.

Reference:  

* [dnf list plugins command missing](https://bugzilla.redhat.com/show_bug.cgi?id=1694041)  

### 3.How to add startup application on CentOS 8?
A:We can add by gnome tweaks. Of course you should install it with Software.

Reference:  

* [Tweaking GNOME Desktop Environment on CentOS 8](https://linuxhint.com/tweaking_gnome_desktop_centos8/)  

### 4.开启 CentOS 8 上 tomcat 9 的注意事项
A:首先是安装 `tomcat-native`；其次是注意从日志文件中定位错误。我遇到了证书文件权限导致找不到文件的情况。

### 5.man: can't set the locale; make sure `$LC_*` and `$LANG` are correct
A:问题是由于 ssh 终端的 locale 设置导致系统的 locale 设置出现问题，我关闭了 sshd_config 中 locale 相关的设置，使用系统的 locale 设置。  

Reference:  

* [Locale](https://wiki.archlinux.org/index.php/Locale#Make_locale_changes_immediate)  
* [How to change system locale on RHEL7?](https://access.redhat.com/solutions/974273)  

### 6.在 CentOS 8 上使用 alternatives 设置默认的 java
A: 在 CentOS 8 上安装 java 包之后不知为什么 alternatives 中的配置居然不对，导致提示 java command not found，于是只好手动配置：  

{% codeblock %}
# 初始情况如下
$ sudo alternatives  --list
libnssckbi.so.x86_64  	auto  	/usr/lib64/pkcs11/p11-kit-trust.so
python                	manual	/usr/bin/python3
ifup                  	auto  	/usr/libexec/nm-ifup
cifs-idmap-plugin     	auto  	/usr/lib64/cifs-utils/cifs_idmap_sss.so
python3               	auto  	/usr/bin/python3.6
nmap                  	auto  	/usr/bin/ncat
java                  	manual	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el8_2.x86_64/jre/bin/java
jre_openjdk           	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el8_2.x86_64/jre
jre_1.8.0             	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.262.b10-0.el8_2.x86_64/jre
jre_1.8.0_openjdk     	auto  	/usr/lib/jvm/jre-1.8.0-openjdk-1.8.0.262.b10-0.el8_2.x86_64
links                 	manual	/usr/bin/elinks
javac                 	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64/bin/javac
java_sdk_openjdk      	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64
java_sdk_1.8.0        	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64
java_sdk_1.8.0_openjdk	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64

# 确认下正确设置的相关参数
$ alternatives --display java_sdk_openjdk
java_sdk_openjdk - status is auto.
 link currently points to /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64 - family java-1.8.0-openjdk.x86_64 priority 1800265
Current `best' version is /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64.

# 设置默认 java 相关功能的路径
sudo alternatives --install /usr/bin/java java /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64/jre/bin/java 1800265 --family java-1.8.0-openjdk.x86_64

sudo alternatives --install /usr/lib/jvm/jre_openjdk jre_openjdk /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64/jre 1800265 --family java-1.8.0-openjdk.x86_64

sudo alternatives --install /usr/lib/jvm/jre_1.8.0 jre_1.8.0 /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64/jre 1800265 --family java-1.8.0-openjdk.x86_64

sudo alternatives --install /usr/lib/jvm/jre_1.8.0_openjdk jre_1.8.0_openjdk /usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64 1800265 --family java-1.8.0-openjdk.x86_64 

# 确认设置结果
$ alternatives --list
libnssckbi.so.x86_64  	auto  	/usr/lib64/pkcs11/p11-kit-trust.so
python                	manual	/usr/bin/python3
ifup                  	auto  	/usr/libexec/nm-ifup
cifs-idmap-plugin     	auto  	/usr/lib64/cifs-utils/cifs_idmap_sss.so
python3               	auto  	/usr/bin/python3.6
nmap                  	auto  	/usr/bin/ncat
links                 	manual	/usr/bin/elinks
javac                 	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64/bin/javac
java_sdk_openjdk      	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64
java_sdk_1.8.0        	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64
java_sdk_1.8.0_openjdk	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64
java                  	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64/jre/bin/java
jre_openjdk           	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64/jre
jre_1.8.0             	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64/jre
jre_1.8.0_openjdk     	auto  	/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64
{% endcodeblock %}

另外我们还可以在 `/etc/profile.d` 目录下新建 `java.sh` 文件来设置 `JAVA_HOME` 和 `JRE_HOME`：  

{% codeblock %}
# /etc/profile.d/java.sh
JAVA_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64"
JRE_HOME="/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.265.b01-0.el8_2.x86_64/jre"
{% endcodeblock %}

Reference:  

* [Introduction to the alternatives command in Linux](https://www.redhat.com/sysadmin/alternatives-command)  

### 7. chsh command not available on CentOS 8
A:

{% codeblock %}
$ dnf provides '*/chsh'
Last metadata expiration check: 8 days, 1:03:13 ago on Thu 01 Oct 2020 03:47:40 AM UTC.
util-linux-user-2.32.1-22.el8.x86_64 : libuser based util-linux utilities
Repo        : BaseOS
Matched from:
Filename    : /etc/pam.d/chsh
Filename    : /usr/bin/chsh
Filename    : /usr/share/bash-completion/completions/chsh

$ rpm -q util-linux-user-2.32.1-22.el8.x86_64
package util-linux-user-2.32.1-22.el8.x86_64 is not installed

$ sudo dnf install util-linux-user-2.32.1-22.el8.x86_64
{% endcodeblock %}

* [CentOS 8.0.1905 - 'chsh' : command not found](https://forums.centos.org/viewtopic.php?t=73864)  

### 8. rsync 同步目录时产生了很多以 `~` 结尾的文件
A:原因是加上了 b 选项，会对文件做备份

{% codeblock %}
rsync -avuzb treasure-workspace/dist/treasure/ treasure-lib-dist
{% endcodeblock %} 

Reference:  

* [What does the tilde (~) mean at the end of a filename? ](https://unix.stackexchange.com/questions/76189/what-does-the-tilde-mean-at-the-end-of-a-filename)  
* [What is bitwise.c~? ](https://unix.stackexchange.com/questions/132669/what-is-bitwise-c?noredirect=1&lq=1)  


