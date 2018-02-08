---
layout: post
title: "Mac 使用笔记(二)"
date: 2018-01-23 11:59:31 +0800
comments: true
categories: [Archives]
keywords: Mac
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

```
// ~/.bash_profile
export var=value
```

## 3.SSH ask passpharse every time I use SSH key.

A:

```
$echo -e "AddKeysToAgent yes\nUseKeychain yes" >> ~/.ssh/config
```

##4.Fix the Enable and Disable install software from anywhere in macOS Sierra
A:

```
// Enable
$ sudo spctl --master-disable

// Disable
$sudo spctl --master-enable
```

Reference:[Fix the Enable and Disable install software from anywhere in macOS Sierra problem](https://www.osxio.com/fix-enable-disable-install-software-anywhere-macos-sierra-problem/)

##5. Mac 上批量替换文件中的字符串
A:

```
$ grep -rl discription source/_posts | xargs sed -i ''  "s/discription/description/g"
```
Reference:[linux 批量替换文件内容及查找某目录下所有包含某字符串的文件（批量修改文件内容）](http://blog.csdn.net/werm520/article/details/49334513)  
[论mac使用sed修改文件的正确姿势](http://xiaorui.cc/2016/01/14/%E8%AE%BAmac%E4%BD%BF%E7%94%A8sed%E4%BF%AE%E6%94%B9%E6%96%87%E4%BB%B6%E7%9A%84%E6%AD%A3%E7%A1%AE%E5%A7%BF%E5%8A%BF/)  


