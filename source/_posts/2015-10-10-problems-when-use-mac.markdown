---
layout: post
title: "使用 Mac 遇到的问题"
date: 2015-10-10 08:49:37 +0800
comments: true
categories: [Archives]
keywords: Mac
discription: 
---

## 1. How to enable the three finger drag on OS X 10.11
Step 1: Open System Preferences

Step 2: Click Accessibility

Step 3: Click Mouse & Trackpad

Step 4: Click Trackpad Options…

Step 5: Click Enable dragging

Step 6: Select “three finger drag” in the drop down box

Reference:http://www.idownloadblog.com/2015/06/25/three-finger-drag-gesture-os-x-el-capitan/
<!-- more -->

## 2. How to find computer name on Mac
1. GUI  
 System Preference > Sharing
 
2. Command Line
	
	```
	$ hostname
	```
	
## 3. SSH to Server Without Entering Password From Mac 

```
$ cp ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
```

Reference:https://thecustomizewindows.com/2014/04/ssh-server-without-entering-password-mac-os-x/

<!-- more -->

## 4. How to open google search results in a new tab？

Solution:Here's how to make Google open your results in a new tab every time. If you'd rather Google not open your results on the same page, you can set the default by going to Google and clicking on the cog in the upper right-hand corner. Choose "Search Settings" from the list, then scroll down to "Where Results Open".

## 5. 怎么输入千分号？
Solution:shift-option-R输入‰
Reference:http://newping.cn/414

## 6. Mac keyboard shortcuts
Solution:
Command ⌘
Shift ⇧
Option ⌥
Control ⌃
Caps Lock ⇪
Fn

Reference:https://support.apple.com/en-us/HT201236

## 7. Add a user to a group 
Solution:

```
$ sudo dscl localhost -append /Local/Default/Groups/thegroupname GroupMembership theusername
```

## 8. To create a group
Solution:

```
$ dscl / -create /Groups/GROUP
$ dscl / -create /Groups/GROUP PrimaryGroupID GID
$ dscl / -create /Groups/GROUP Password \*
```

## 8. To create a user
Solution:

```
$ dscl / -create /Users/USER
$ dscl / -create /Users/USER UniqueID UID
$	dscl / -create /Users/USER UserShell /usr/bin/false
$	dscl / -create /Users/USER RealName 'DESCRIPTION'
$	dscl / -create /Users/USER NFSHomeDirectory DIRECTORY
$	dscl / -create /Users/USER PrimaryGroupID GID
$	dscl / -create /Users/USER Password \*
```

## 9. How to view whether port was occupied under mac?

```
1) netstat -an | grep 'port'
2) lsof -i:port
```

## 10. If you forgot the passcode for your iPhone, iPad, or iPod touch, or your device is disabled
A:https://support.apple.com/en-us/HT204306

## 11. Disable AAM Updates Notifier

```
$ sudo mv /Library/LaunchAgents/com.adobe.AAM.Updater-1.0.plist /Library/LaunchAgents/com.adobe.AAM.Updater-1.0.plist.disable
```

## 12. 如何解决Mac里面解压后文件名乱码问题?
A: http://www.jayz.me/?p=493


