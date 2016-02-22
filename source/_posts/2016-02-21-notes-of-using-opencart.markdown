---
layout: post
title: "Notes Of Using OpenCart"
date: 2016-02-21 17:50:35 +0800
comments: true
categories: [Archives, Web Development]
keywords: OpenCart
discription: Notes Of Using OpenCart
---
###1. opencart 安装中文语言包
A:

* [下载中文语言包](http://www.opencart.cn/thread-9982-1-1.html）
* 解压压缩包，里面有两个文件夹，admin是后台语言包，catalog文件是前台语言包
* 把文件上传到对应的目录下

```
$ cd ~/Downloads/
$ tar -xzvf OpenCart_SimplifiedChinese_language_v2.0.3.1.zip
$ cp -R OpenCart_SimplifiedChinese_language_v2.0.3.1/upload/catalog/language/zh-CN  ~/Sites/upload/catalog/language/
$ cp -R OpenCart_SimplifiedChinese_language_v2.0.3.1/upload/admin/language/zh-CN ~/Sites/upload/admin/language/
```

* System > Localisation > languages > + > Chinese(Language Name) > Code(select zh-CN) > Save
* System > Settings > Your Store > Edit > Local > Language (Chinese) > Administration Language (Chinese) > Save

###2. opencart如何增加人民币？
A:

* 系统设置 > 参数设置 > 货币设置 > + 
	
	* 货币名称 人民币
	* 货币代码 CNY
	* 左符号 ¥
	* 右符号
	* 小数位 2
	* 汇率 1.00000
	* 状态 启用

* 系统设置 > 网店设置 > 选择你的网店 > 编辑 > 本地化设置 > 货币设置 > 人民币 > 保存

Reference:http://www.opencart-extension.com/opencart-add-renminbi.html

###3. 如何添加推荐商品？
A: 扩展功能 > 模块配置 > 推荐商品 > Home Page > Product Name

###4. 如何设置幻灯片？
A: 设计 > 横幅 > Home Page Slideshow > Edit


