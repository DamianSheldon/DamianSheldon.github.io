---
layout: post
title: "Mac 使用笔记(二)"
date: 2018-01-23 11:59:31 +0800
comments: true
categories: [Archives]
keywords: Mac, spctl, ssh, sed, lsof
description: 使用 Mac 时遇到的问题
---

## 1. iPhone每次连接Mac都会弹出iPhoto
解决办法：

1. Plug in your iPad/iPhone
2. Open Image Capture
3. Select your device ("devMikePad")
4. Press the triangle in square symbol in the lower left corner.
5. Select "No application" in the menu.

iPhoto:
Preferences > General > "Connecting Camera Opens ..."
set it to "No Application"

## 2. Add environment variable

```bash
// ~/.bash_profile
export var=value
```

## 3.SSH ask passpharse every time I use SSH key.

A:

```bash
$echo -e "AddKeysToAgent yes\nUseKeychain yes" >> ~/.ssh/config
```

##4.Fix the Enable and Disable install software from anywhere in macOS Sierra
A:

```bash
// Enable
$ sudo spctl --master-disable

// Disable
$sudo spctl --master-enable
```

Reference:[Fix the Enable and Disable install software from anywhere in macOS Sierra problem](https://www.osxio.com/fix-enable-disable-install-software-anywhere-macos-sierra-problem/)

##5.Mac 上批量替换文件中的字符串
A:

```bash
$ grep -rl discription source/_posts | xargs sed -i ''  "s/discription/description/g"
```
Reference:[linux 批量替换文件内容及查找某目录下所有包含某字符串的文件（批量修改文件内容）](http://blog.csdn.net/werm520/article/details/49334513)  
[论mac使用sed修改文件的正确姿势](http://xiaorui.cc/2016/01/14/%E8%AE%BAmac%E4%BD%BF%E7%94%A8sed%E4%BF%AE%E6%94%B9%E6%96%87%E4%BB%B6%E7%9A%84%E6%AD%A3%E7%A1%AE%E5%A7%BF%E5%8A%BF/)  

##6.如何查看占用指定端口的进程？  
A:

```bash
$ lsof -i :[port]
```

结果输出中有进程的 pid, 之后我们可以用这个 pid 来杀掉该进程：

```bash
$ kill -9 [pid]
```

这些进程很可能是系统启动时就启动了，可以通过：

```bash
$ launchctl list | grep [pid]
```

来确认。如果找到有，而我们又不想它自启动了，可以在以下目录中找到配置文件：

```
~/Library/LaunchAgents         Per-user agents provided by the user.
/Library/LaunchAgents          Per-user agents provided by the administrator.
/Library/LaunchDaemons         System wide daemons provided by the administrator.
```

找到之后，可以用命令：  

```bash
$ launchctl unload -w paths
// eg:
$ launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
```

查看启动进程的命令:  

```
ps -p <pid> -wwf
```

<!--more-->
##7.如何使用命令行安装 dmg？
A:

First, mount the dmg image : `sudo hdiutil attach <image>.dmg`  

The image will be mounted to /Volumes/<image>. Then we can install with: `sudo installer -package /Volumes/<image>/<image>.pkg -target /`  

Finally unmount the image: `sudo hdiutil detach /Volumes/<image>`.  

这个需求是源于最近一次安装 mysql 时，图形界面安装无响应，之后尝试命令行成功安装，还蛮奇怪。  

Reference:[Is there a command to install a dmg](https://apple.stackexchange.com/questions/73926/is-there-a-command-to-install-a-dmg)  

##8.How to start Mac's built-in SMTP server?
A:Mac include a built-in SMTP server, but its configuration on Mojave seems not property for daemon. Because KeepAlive not set to true, so we have to correct it. However its configuration file locate at `/System/Library/LaunchDaemons/org.postfix.master.plist`, we can't edit it even as root, SIP only can diable on recovery OS on mojave. Eventually I think a workaround, detail as follow:

{% codeblock %}

$ sudo launchctl unload -w /System/Library/LaunchDaemons/org.postfix.master.plist
$ sudo mv /System/Library/LaunchDaemons/org.postfix.master.plist /Library/LaunchDaemons/org.postfix.master.plist  

// Edit plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>org.postfix.master</string>
	<key>Program</key>
	<string>/usr/libexec/postfix/master</string>
	<key>ProgramArguments</key>
	<array>
		<string>master</string>
		<string>-e</string>
		<string>60</string>
	</array>
	<key>QueueDirectories</key>
	<array>
		<string>/var/spool/postfix/maildrop</string>
	</array>
	<key>AbandonProcessGroup</key>
	<true/>
    <key>RunAtLoad</key>
    <true/>
    <key>KeepAlive</key>
    <true/>
</dict>
</plist>

// Check plist syntax 
$ plutil -lint /Library/LaunchDaemons/org.postfix.master.plist

// Load service
$ sudo launchctl load -w /Library/LaunchDaemons/org.postfix.master.plist

// Check service
$ sudo lsof -i :25

// Or 
$ telnet localhost 25
{% endcodeblock %}

##9.从“时间机器”备份恢复文件
A: Apple 介绍了如何从 “时间机器”备份恢复文件，你可以用备份创建一个新的用户，它拥有备份时状态。但我的情况是备份之后还在继续使用，产生了新的文件，想恢复那些删除的文件，但不覆盖新的文件，而且由于电脑的硬盘较小，我直接用备份创建一个新的用户的恢复方法空间不够。于是我只能手动从“时间机器”的备份里恢复文件。  

1. Enable Full Disk Access for your terminal, I use iTerm;
	
	* System Preferences > Security & Privacy > Privacy > Full Disk Access > Add iTerm
	
2. Use rsync restore;
	
	* eg `$rsync -avuzb Documents ~/`

Reference:  

* [从备份恢复 Mac](https://support.apple.com/zh-cn/HT203981)  
* [How can change broken file permissions of Time Machine backups?](https://apple.stackexchange.com/questions/365062/how-can-change-broken-file-permissions-of-time-machine-backups)  

##10.升级到 macOS Catalina 后，根目录下的非标准目录丢失了
A:

> While creating the two separate volumes during the upgrade process, files and data that couldn’t be moved to their new location are placed in a Relocated Items folder. The Relocated Items folder is in the Shared folder within the User folder (/Users/Shared/Relocated Items) and available though a shortcut on the Desktop.

我之前是将私有 git 仓库目录放在了 `/git`， 所以现在需要使用 rsync 恢复到新的目录: `rsync -avuzb /Users/Shared/Relocated\ Items/Security/git/ repo/`

Reference:  

* [Where does the upgrade to macOS Catalina move root “/” directory files?](https://apple.stackexchange.com/questions/371852/where-does-the-upgrade-to-macos-catalina-move-root-directory-files)  
* [About the read-only system volume in macOS Catalina](https://support.apple.com/en-us/HT210650)  

