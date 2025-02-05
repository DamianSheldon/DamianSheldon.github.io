---
layout: post
title: "iTunes 歌曲信息乱码的解决办法"
date: 2014-11-03 10:23:29 +0800
comments: true
categories: [Archives]
keywords: Garbled, 乱码, iTunes, Music
description: Fix Music that import from windows garbled text in iTunes
---

从Windows迁移过来的歌曲在iTunes中乱码显示，按照Apple support里面的方法添加语言并没有解决问题，因此，我推测问题应该不是出在这。经过查找，发现问题是由于歌曲的ID3中的Enconding by字段的影响，也就是说，编码格式不一样。解决办法：   
1)下载[Mutagen](https://bitbucket.org/lazka/mutagen);   
2)安装Mutagen;   
```bash
    $ path_to_mutagen/setup.py build
    $ sudo path_to_mutagen/setup.py install
```
3)先将所有歌曲备份，防止操作出错；  
4)将目录下的所有MP3歌曲的编码转成Unicode;  
```bash
    $find . -iname "*.mp3" -execdir mid3iconv -e gbk {} \;  
```
5)将iTunes中的音乐清空，重新添加。  

Reference:http://floss.zoomquiet.io/data/20070510235547/index.html  
