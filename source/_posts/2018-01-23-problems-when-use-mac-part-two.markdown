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

##7.如何使用命令行安装 dmg？
A:

First, mount the dmg image : `sudo hdiutil attach <image>.dmg`  

The image will be mounted to /Volumes/<image>. Then we can install with: `sudo installer -package /Volumes/<image>/<image>.pkg -target /`  

Finally unmount the image: `sudo hdiutil detach /Volumes/<image>`.  

这个需求是源于最近一次安装 mysql 时，图形界面安装无响应，之后尝试命令行成功安装，还蛮奇怪。  

Reference:[Is there a command to install a dmg](https://apple.stackexchange.com/questions/73926/is-there-a-command-to-install-a-dmg)  


