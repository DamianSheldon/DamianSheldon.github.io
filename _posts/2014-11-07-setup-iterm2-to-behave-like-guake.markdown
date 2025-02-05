---
layout: post
title: "为MacBook Pro配置一个像guake的终端"
date: 2014-11-07 10:31:19 +0800
comments: true
categories: [Archives]
keywords: iterm2, guake, mac
description: setup iterm2 to behave like guake
---
以前在Arch下经常使用一个名为guake的终端，体验很好，用MacBook Pro之后也想有个这样的终端，由于各种原因，这事一直耽搁着，但并没有放下，今天特意找了下，还真找到了。不费话了，进入正题。

###让iTerm2的行为像guake   

1)下载[iTerm2](http://iterm2.com/),然后解压;    

2)打开iTerm2,然后iTerm2-->Prefences...->Profiles;     

3)创建一个新Profiles,命名为guake;   

4)在Window选项中根据自己的喜好配置;     

<img name="create_new_iterm_profile" src="/images/create_new_iterm_profile.png" width="913" height="533">  

5)在Keys选项中激活"Show/hide iTerm2 with a system-wide hotkey",由于F12用来移动到Dashboard,只能用其他的快捷键了，可以根据自己喜好设置，我这里设置为⌘F12,(Note:⌘ + fn + F12);     

<img name="assign_a_hotkey" src="/images/assign_a_hotkey.png" width="897" height="528">  

<!-- more -->

###开机启动iTerm2时不打开终端窗口
安装好iTerm2,并把它配置像guake, 加入开机自启动(System Preferences > Users&Groups > Login Items > + iTerm2)之后，还有一个小问题困扰着我，就是它会默认打开一个终端窗口，这让人很不舒坦，解决方法如下:     

1)打开iTerm2;    

2)关闭所有的窗口(iTerm2菜单栏-->shell-->Close);   

3)Window-->Save Window Arrangement;   

4)将新的窗口布局命名为“No Windows”;  

5)将这个新窗口布局设置默认布局，Preferences > Arrangements > Set it as default;   

6)最后在Preferences… > General > Startup, 只选中“Open default window arrangement” 。

###Reference:   
[SETUP ITERM2 TO BEHAVE LIKE GUAKE](http://ivanvillareal.com/osx/setup-iterm2-to-behave-like-guake/)    

[Launch iTerm 2 on startup without opening a terminal window](http://rottmann.net/2013/03/launch-iterm-2-on-startup-without-opening-a-terminal-window/)

