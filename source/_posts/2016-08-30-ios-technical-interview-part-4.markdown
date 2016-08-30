---
layout: post
title: "iOS Technical Interview Part 4"
date: 2016-08-30 10:43:36 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS Technical Interview
discription: iOS Technical Interview  
---

###1.熟悉 CocoaPods 么？能大概讲一下工作原理么？
A:熟悉，项目中的 Podfile 文件指定了使用的第三方库，根据它可以找到对应的 podspec 。 podspec 文件存储着第三方库的基本信息：包含哪些文件，静态库，资源文件，信赖哪些其他第三方库、系统框架，之后 pod 会创建一个工程，把这个库以 target 的形式包含进来，应用则信赖这个 pod 工程。

###2.最常用的版本控制工具是什么，能大概讲讲原理么？
A: Git

###3.你一般是怎么用 Instruments 的？
A:根据想要分析的问题，选择对应的　instruments　，同时参考　Instruments　User　Guide．

###4.你一般是如何调试 Bug 的
A:复现 bug，之后单步调试定位。

###5.你在你的项目中用到了哪些设计模式？
A: 

	* 单例
	* 委托
	* Target-Action
	* MVC
	* 观察者
	* 工厂方法
	* 信息流
	* 类簇

###6.如何实现单例，单例会有什么弊端？
A：

```
+ (instancetype)sharedInstance
{
    static dispatch_once_t once;
    static id sharedInstance;
    dispatch_once(&once, ^{
        sharedInstance = [[self alloc] init];
    });
    return sharedInstance;
}
```

* 单例相当于全局变量，因此存在强藕合，不利于扩展和应对变化；
* 单例违背单一设计原则；

###7.如何把一个包含自定义对象的数组序列化到磁盘？
A:自定义对象需要实现 NSCoding 协议，然后可以调用 NSKeyedArchiver 的`+ (BOOL)archiveRootObject:(id)rootObject toFile:(NSString *)path`方法将它序列化到磁盘。

###8.iOS 的沙盒目录结构是怎样的？ App Bundle 里面都有什么？
A:

App Bundle 里面有 Info.plist, Executeable, Resource files and Other support files.


###9.iOS 7的多任务添加了哪两个新的 API? 各自的使用场景是什么？
A:

* Apps that regularly update their content by contacting a server can register with the system and be launched periodically to retrieve that content in the background. To register, include the UIBackgroundModes key with the fetch value in your app’s Info.plist file. Then, when your app is launched, call the setMinimumBackgroundFetchInterval: method to determine how often it receives update messages. Finally, you must also implement the application:performFetchWithCompletionHandler: method in your app delegate.
* Apps that use push notifications to notify the user that new content is available can fetch the content in the background. To support this mode, include the UIBackgroundModes key with the remote-notification value in your app’s Info.plist file. You must also implement the application:didReceiveRemoteNotification:fetchCompletionHandler: method in your app delegate.

Reference: What's New in iOS

###10.Objective-C 的 class 是如何实现的？Selector 是如何被转化为 C 语言的函数调用的？

###11.UIScrollView 大概是如何实现的，它是如何捕捉、响应手势的？

###12.Objective-C 如何对已有的方法，添加自己的功能代码以实现类似记录日志这样的功能？
A: Method Swizzle.

###13.`+load` 和 `+initialize` 的区别是什么？
A:

* load:Invoked whenever a class or category is added to the Objective-C runtime; implement this method to perform class-specific behavior upon loading.
* initialize:Initializes the class before it receives its first message.

###14.NSOperation 相比于 GCD 有哪些优势？
A:

* 可以取消操作；
* 可以添加依赖；

###15.strong / weak / unsafe_unretained 的区别

A: 

* strong:对象的引用计数加一;  
* weak:对象的引用计数不变，对象销毁时 weak 属性自动置为nil;  
* unsafe_unretained:对象的引用计数不变，对象销毁时 unsafe_unretained 属性成为野指针;  

你可能会奇怪，有了 weak 还要 unsafe_unretained 干什么？ 原因是 weak 是 iOS 5 才可用的，所以当你要兼容 iOS 5, 或者将 iOS 5 时代之前 MRC 代码迁移到 ARC 时会用它，除些之外我们应该使用 weak.

Reference:http://stackoverflow.com/questions/15968198/what-is-the-use-of-unsafe-unretained-attribute

###16.Objective-C 中，meta-class 指的是什么

###17.UIView 和 CALayer 之间的关系？
A: UIView 会持有至少一个 CALayer 实例，CALayer 是 UIView 的骨架，它将 UIView 的内容绘制出来并提供动画支持。

###18.`+[UIView animateWithDuration:animations:completion:]` 内部大概是如何实现的？
A:

###19.什么时候会发生「隐式动画」？
A: CALayer 的实例是支持隐式动画的，所以修改 CALayer Animatable Properties 里面的属性都可以触发隐式动画。UIView 默认禁止了 CALayer 的隐式动画，在动画块中又使能了，所以和 UIView 关联的 CALayer 实例只能在动画块中修改 Animatable UIView properties 里的属性也可以触发隐式动画。

Reference:http://stackoverflow.com/questions/4749343/when-exactly-do-implicit-animations-take-place-in-ios

###如何把一张大图缩小为1/4大小的缩略图？

###21.Toll-Free Bridging 是什么？什么情况下会使用？

###22.当系统出现内存警告时会发生什么？

###23.什么是 Protocol，Delegate 一般是怎么用的？

###24.UIWebView 有哪些性能问题？有没有可替代的方案。

###25.为什么 NotificationCenter 要 removeObserver? 如何实现自动 remove?

###26.当 TableView 的 Cell 改变时，如何让这些改变以动画的形式呈现？

###27.什么是 Method Swizzle，什么情况下会使用？

###28.为什么当 Core Animation 完成时，layer 又会恢复到原先的状态？

###29.你会如何存储用户的一些敏感信息，如登录的 token。

###30.什么时候会使用 Core Graphics，有什么注意事项么？

###31.使用 Block 时需要注意哪些问题？

###32.performSelector:withObject:afterDelay: 内部大概是怎么实现的，有什么注意事项么？

###33.如何播放 GIF 图片，有什么优化方案么？

###34.有哪几种方式可以对图片进行缩放，使用 CoreGraphics 缩放时有什么注意事项？

###35.哪些途径可以让 ViewController 瘦下来？

###36.有哪些常见的 Crash 场景？

###37.设计一套大文件（如上百M的视频）下载方案。

###38.如果让你来实现 dispatch_once，你会怎么做？


Reference:https://github.com/lzyy/iOS-Developer-Interview-Questions

