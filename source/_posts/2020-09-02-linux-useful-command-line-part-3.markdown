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
rsync -avuzb /tmp/etc/ /etc
rsync -avuzb /tmp/home/ /home
rsync -avuzb /tmp/var/spool/mail/ /var/spool/mail
rsync -avuzb /tmp/var/spool/cron/ /var/spool/cron
rsync -avuzb /tmp/root/ /root
rsync -avuzb /tmp/usr/local/bin/ /usr/local/bin
```
<!--more-->
#### 系统迁移
换新电脑或者更换云服务时我们可能不想要上面那么麻烦，而可能想直接迁移系统，至少我是这么想的，这时我们可以使用下面的方法：

```
# 全系统备份
tar --create --absolute-names --preserve-permissions --bzip2  --file=/media/sf_Windows10-shared-folder/virtual-box-centos-8.tar.bz2 --exclude=/dev --exclude=/media --exclude=/metainfo --exclude=/mnt --exclude=/proc --exclude=/run --exclude=/sys  --exclude=/tmp --exclude=/var --exclude=/@System.solv /

# 如有需要也可检查备份的文件
tar -tjpPvf /media/sf_Windows10-shared-folder/virtual-box-centos-8.tar.bz2 | less

# 将备份包放到 /tmp 下解压
tar -xjvf virtual-box-centos-8.tar.bz2
```

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

