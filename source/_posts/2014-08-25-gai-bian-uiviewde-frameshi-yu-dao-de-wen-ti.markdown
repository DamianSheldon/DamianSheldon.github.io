---
layout: post
title: "iOS App 开发问题汇总（持续更新）"
date: 2014-08-25 17:09:49 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS, UIView, Frame， Switch, Case
description: iOS App 开发问题汇总
---

###问题描述：Storyboard中的ViewController上添加一个自定义的view,声明为IBOutlet然后用代码改变view的Frame,打印输出Frame的值确实改变了，但是模拟器上的视图的Frame还是没有改变。

  解决办法：Google找到Stackoverflow上有人说是选中了Auto layout的原因，取消之后确实生效了。PS:但是不知道问题的原因是什么。

Reference:  
o http://stackoverflow.com/questions/18263359/setting-the-frame-of-an-uiview-does-not-work

###问题描述：在switch语句中，如果在case中要定义变量的话要加上大括号。

原因：Case statements are only 'labels'. This means the compiler will interpret this as a jump directly to the label.The problem here is one of scope. Your curly brackets define the scope as everything inside the 'switch' statement. This means that you are left with a scope where a jump will be performed further into the code skipping the initialization. The correct way to handle this is to define a scope specific to that case statement and define your variable within it.  

Reference:http://stackoverflow.com/questions/92396/why-cant-variables-be-declared-in-a-switch-statement/92439#92439

###问题描述：创建Group style的UITalbeView的顶端有很大一块空白。  

解决办法：YouStoryboard.storyboard > YouViewController > Attributes inspector > Uncheck - Adjust scroll view insets    

Reference:http://stackoverflow.com/questions/18880341/why-is-there-extra-padding-at-the-top-of-my-uitableview-with-style-uitableviewst   

<!-- more -->

###问题描述：SVN更新Cocoapods管理的第三方包的Xcode工程报错。    
```bash
A  +  C Pods
>   local edit, incoming delete upon update
```

解决办法：svn revert --depth infinity Pods  

Reference:http://stackoverflow.com/questions/4317973/svn-how-to-resolve-local-edit-incoming-delete-upon-update-message  

###问题描述：local unversioned, incoming add upon update

```
$svn status
D     C removed_directory
>   local unversioned, incoming add upon update
Summary of conflicts:
Tree conflicts: 1
```

解决办法:

```
$ svn resolve --accept working removed_directory
Resolved conflicted state of 'removed_directory'
$ svn revert removed_directory
Reverted 'removed_directory'
$ svn status
$
```


###问题描述：*** Terminating app due to uncaught exception 'NSInternalInconsistencyException', reason: '-[UIViewController _loadViewFromNibNamed:bundle:] loaded the "loc" nib but the view outlet was not set.'    

解决办法：从输出的日志来看，是说 view 没有设置，事实也确实如此。我先创建了 UIViewController 类文件，然后再创建一个空的同名 xib 文件，我从 Object Library 中拉了一个 UIViewController，问题就出在这里，我应该拉一个 UIView ，并将 File's Owner 设置成正确的类名，最后将 view outlet 联接起来。所以，如果想用 xib 创建 UIViewController，建议在创建类的时候勾选创建相应的 Xib 文件，让 Xcode 做好这些工作。   

Reference:http://www.cnblogs.com/tivonstone/archive/2012/04/20/2460116.html    

###问题描述：设置swipe gesture的direction为UISwipeGestureRecognizerDirectionLeft | UISwipeGestureRecognizerDirectionRight但是只识别一个方向。   

解决办法：为每个方向单独创建一个UISwipeGestureRecognizer。   

Reference:http://stackoverflow.com/questions/7420078/detect-when-uigesturerecognizer-is-up-down-left-and-right-cocos2d/7760927#7760927    


###问题描述：应用获取用户位置信息弹出框很快自动消失

解决办法：问题的原因是CLLocationManager实例是局部变量，出了作用域很快会被销毁。因此只要将CLLocationManager实例声明为类的property或私有实例变量就能解决。

Reference:http://stackoverflow.com/questions/7888896/current-location-permission-dialog-disappears-too-quickly 

###问题描述：位置服务在iOS 8上失效
解决办法：在plist配置文件中增加NSLocationAlwaysUsageDescription 或NSLocationWhenInUseUsageDescription，然后在代码中调用requestAlwaysAuthorization 或 requestWhenInUseAuthorization。NSLocationAlwaysUsageDescription的描述见Information Property List Key Reference。
Reference:http://stackoverflow.com/questions/24062509/location-services-not-working-in-ios-8

###问题描述：如何查看Framework支持哪些架构
解决办法：

```
// View RWUIControls.framework支持的架构
$ cd ~/Desktop/RWUIControls.framework
$ RWUIControls.framework  xcrun lipo -info RWUIControls
```

###问题描述：when install Photoshop CS recently I got an error message saying that case-sensitive file system cannot be used for installation.
解决办法：
I converted the default case-sensitive HFS+ partition to a case insensitive one after discovering the problem after installing a new MacBook. I assume here that you have enough disk space on your internal hard drive to duplicate the data and system files that you already have installed.

* Use Disk Utility to shrink the size of your existing boot partition to just big enough to contain the existing files.
* Create a new partition that is only Mac OS (Journaled) and is NOT case sensitive.
* Backup the original drive to the new partition. I used SuperDuper! but you can use rsync.
* Boot holding down the Option key and select the new partition.
* (Option)Delete the old partition with Disk Utility and increase the size of the new one.

Reference:http://apple.stackexchange.com/questions/15080/convert-a-partition-from-case-sensitive-to-case-insensitive

####问题描述：2015-01-14 08:42:44.898 cstApp[5711:2038701] *** Terminating app due to uncaught exception 'NSGenericException', reason: '*** Collection <__NSArrayM: 0x174253710> was mutated while being enumerated.'
*** First throw call stack:
(0x18734259c 0x197a500e4 0x187341f50 0x1002ebd00 0x1001e8e94 0x1001e9e90 0x18bb28d34 0x18bb11e48 0x18bb286d0 0x18bb2835c 0x18bb218b0 0x18baf4fa8 0x18bd93f58 0x18baf3510 0x1872fa9ec 0x1872f9c90 0x1872f7d40 0x1872250a4 0x1903cf5a4 0x18bb5a3c0 0x100220370 0x1980bea08)
libc++abi.dylib: terminating with uncaught exception of type NSException

解决办法：问题的原因是调用百度地图的removeAnnotations:方法，我传入调用地图返回的annotations,估计百度底下直接用它进行遍历加删除就出错了。

```
    NSArray *tempArray = [self.bmapView.annotations copy];
    for (BMKPointAnnotation *annotation in tempArray) {
        [self.bmapView removeAnnotation:annotation];
    }
```

