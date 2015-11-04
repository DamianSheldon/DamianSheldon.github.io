---
layout: post
title: "Mac 安装 OS X El Capitan"
date: 2015-10-09 18:00:22 +0800
comments: true
categories: [Archives]
keywords: OS El Capitan, Mac 
discription: 
---
最近 Apple 更新了 OS X，通过 App Store 更新下载下来 OS X El Capitan 的安装时遇到问题了 -- 手头的这台 Mac 之前有人分区安装了 Windows, OS X 安装在第二个分区， OS X El Capitan 却不能安装在 OS X 所在的分区，而且格式化 Windows 所在的分区又报错，也是醉了，只好重装。

## 备份数据  
我用 sftp 把数据备份到另一台 Mac 上, 之前也试过 Migration Assitant 都是卡在最后一分钟，不知道现在这个问题改善了没有。 用 sftp 备份中间也中断了好多次，但手头也没移动硬盘也只能这么将就着了。

## Install OS X El Capitan  
1. 需要一个 8GB 左右的 U 盘，因为 El Capitan 的大小超过 6GB ;
2. 从 App Store 下载 El Capitan;
3. 制作安装盘：
```
sudo /Applications/Install\ OS\ X\ Mavericks.app/Contents/Resources/createinstallmedia --volume /Volumes/Untitled --applicationpath /Applications/Install\ OS\ X\ Mavericks.app --nointeraction
```
4. 重新启动，按住 option 键，选择通过U盘启动，之后先进入磁盘工具里面把硬盘格式化了（你也可以选择不格式化，这样就是覆盖安装）;
<!-- more -->
## 安装常用软件
1. Download Xcode in App Store;
2. iTerm;
3. Chrome;
4. LibreOffice;
5. Microsoft Remote Desktop Connection;
6. Homebrew;
7. ruby;
8. Set up web development envrionment


Reference:http://blog.devtang.com/blog/2014/04/12/install-mavericks-note/

