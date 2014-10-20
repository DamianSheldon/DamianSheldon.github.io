---
layout: post
title: "iOS App 开发遇到的问题"
date: 2014-08-25 17:09:49 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS, UIView, Frame， Switch, Case
description: 总结iOS App 开发遇到的各种问题
---
  问题描述：Storyboard中的ViewController上添加一个自定义的view,声明为IBOutlet然后用代码改变view的Frame,打印输出Frame的值确实改变了，但是模拟器上的视图的Frame还是没有改变。

  解决办法：Google找到Stackoverflow上有人说是选中了Auto layout的原因，取消之后确实生效了。PS:但是不知道问题的原因是什么。

Reference:  
o http://stackoverflow.com/questions/18263359/setting-the-frame-of-an-uiview-does-not-work

问题描述：在switch语句中，如果在case中要定义变量的话要加上大括号。

原因：Case statements are only 'labels'. This means the compiler will interpret this as a jump directly to the label.The problem here is one of scope. Your curly brackets define the scope as everything inside the 'switch' statement. This means that you are left with a scope where a jump will be performed further into the code skipping the initialization. The correct way to handle this is to define a scope specific to that case statement and define your variable within it.  

Reference:http://stackoverflow.com/questions/92396/why-cant-variables-be-declared-in-a-switch-statement/92439#92439

问题描述：创建 Group style 的 UITalbeView 的顶端有很大一块空白。  

解决办法：YouStoryboard.storyboard > YouViewController > Attributes inspector > Uncheck - Adjust scroll view insets    

Reference:http://stackoverflow.com/questions/18880341/why-is-there-extra-padding-at-the-top-of-my-uitableview-with-style-uitableviewst
