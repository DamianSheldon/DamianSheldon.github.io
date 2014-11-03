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

问题描述：创建Group style的UITalbeView的顶端有很大一块空白。  

解决办法：YouStoryboard.storyboard > YouViewController > Attributes inspector > Uncheck - Adjust scroll view insets    

Reference:http://stackoverflow.com/questions/18880341/why-is-there-extra-padding-at-the-top-of-my-uitableview-with-style-uitableviewst   

问题描述：SVN更新Cocoapods管理的第三方包的Xcode工程报错。    
```bash
A  +  C Pods
>   local edit, incoming delete upon update
```

解决办法：svn revert --depth infinity Pods  

Reference:http://stackoverflow.com/questions/4317973/svn-how-to-resolve-local-edit-incoming-delete-upon-update-message  

问题描述：*** Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: '-[UIViewController _loadViewFromNibNamed:bundle:] loaded the "loc" nib but the view outlet was not set.'    

解决办法：从输出的日志来看，是说 view 没有设置，事实也确实如此。我先创建了 UIViewController 类文件，然后再创建一个空的同名 xib 文件，我从 Object Library 中拉了一个 UIViewController，问题就出在这里，我应该拉一个 UIView ，并将 File's Owner 设置成正确的类名，最后将 view outlet 联接起来。所以，如果想用 xib 创建 UIViewController，建议在创建类的时候勾选创建相应的 Xib 文件，让 Xcode 做好这些工作。   

Reference:http://www.cnblogs.com/tivonstone/archive/2012/04/20/2460116.html    

问题描述：设置swipe gesture的direction为UISwipeGestureRecognizerDirectionLeft | UISwipeGestureRecognizerDirectionRight但是只识别一个方向。   

解决办法：为每个方向单独创建一个UISwipeGestureRecognizer。   

Reference:http://stackoverflow.com/questions/7420078/detect-when-uigesturerecognizer-is-up-down-left-and-right-cocos2d/7760927#7760927    


