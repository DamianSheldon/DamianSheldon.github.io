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
A:熟悉，项目中的 Podfile 文件指定了使用的第三方库，根据它可以找到对应的 podspec 。 podspec 文件存储着第三方库的基本信息：包含哪些文件，静态库，资源文件，信赖哪些其他第三方库、系统框架，之后 pod 会创建一个工程，把这个库以 target 的形式包含进来，应用则依赖这个 pod 工程。

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

<!-- more -->

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
A:我觉得`+[UIView animateWithDuration:animations:completion:]` 内部应该是先使能 CALayer 实例的隐式动画，之后对 UIView animatable properties 的设置会传递到 CALayer 对应属性的 setter 方法，CALayer 的 setter 方法会触发对应的隐式动画，最后禁止 CALayer 实例的隐式动画。

Reference:

* https://www.quora.com/How-is-UIViews-+animateWithDuration-animations-implemented   
* http://stackoverflow.com/questions/15175750/how-uiview-animations-with-blocks-work-under-the-hood

###19.什么时候会发生「隐式动画」？
A: CALayer 的实例是支持隐式动画的，所以修改 CALayer Animatable Properties 里面的属性都可以触发隐式动画。UIView 默认禁止了 CALayer 的隐式动画，在动画块中又使能了，所以和 UIView 关联的 CALayer 实例只能在动画块中修改 Animatable UIView properties 里的属性也可以触发隐式动画。

Reference:http://stackoverflow.com/questions/4749343/when-exactly-do-implicit-animations-take-place-in-ios

###20.如何把一张大图缩小为1/4大小的缩略图？
A:

* UIGraphicsBeginImageContextWithOptions & UIImage -drawInRect:(UIKit)
* CGBitmapContextCreate & CGContextDrawImage(Core Graphics)
* CGImageSourceCreateThumbnailAtIndex(Image IO)
* Lanczos Resampling with Core Image(Core Image)
* vImage in Accelerate(vImage)

Reference:http://nshipster.com/image-resizing/

###21.Toll-Free Bridging 是什么？什么情况下会使用？
A:
> There are a number of data types in the Core Foundation framework and the Foundation framework that can be used interchangeably. Data types that can be used interchangeably are also referred to as toll-free bridged data types. This means that you can use the same data structure as the argument to a Core Foundation function call or as the receiver of an Objective-C message invocation.

###22.当系统出现内存警告时会发生什么？
A:系统会杀死后台内存占用量大的应用释放内存给当前运行的应用，当前的应用也会收到内存警告的通知。

###23.什么是 Protocol，Delegate 一般是怎么用的？
A:Protocol是一份消息合约。Delegate 是我们改变一个对象行为方式，这种方式不需要对类做继承。

###24.UIWebView 有哪些性能问题？有没有可替代的方案。
A:

* **线程阻塞问题**--当在 native 层调用 stringByEvaluatingJavaScriptFromString 方法时，可能由于 javascript 是单线程的原因，会阻塞原有 js 代码的执行。这里我们的解决办法是在 js 端用 defer 将 iframe 的插入延后执行。
* **主线程的问题**--UIWebView 的 stringByEvaluatingJavaScriptFromString 方法必须是主线程中执行，而主线程的执行时间过长就会 block UI 的更新。所以我们应该尽量让 stringByEvaluatingJavaScriptFromString 方法执行的时间短。

Reference:[关于UIWebView的总结](http://blog.devtang.com/2012/03/24/talk-about-uiwebview-and-phonegap/)

###25.为什么 NotificationCenter 要 removeObserver? 如何实现自动 remove?
A:因为之后产生了 Observer 感兴趣的通知 NotificationCenter 会调用 Observer 对应的处理方法，如果 Observer 销毁之后不从 NotificationCenter remove,那么会导致应用崩溃。

想要实现自动 remove，如果 NotificationCenter 对 Observer 的引用在 Observer 销毁后能自动置为 nil，类似 weak 的效果，那么问题就解决了。我们不能去修改 NotificationCenter 的内部实现，所以只能用其他的办法。有一个想法是在 Observer 销毁时调用 remove，可以利用 Objective-C 的 runtime 来帮助我们实现。具体思路是混写 addObserver 的方法，在这个混写方法中创建一个对象和 Observer 的生命周期关联起来，然后在这个关联对象的销毁方法中调用 removeObserver。

Reference:[Automatic removal of NSNotificationCenter or KVO observers](http://merowing.info/2012/03/automatic-removal-of-nsnotificationcenter-or-kvo-observers/)

###26.当 TableView 的 Cell 改变时，如何让这些改变以动画的形式呈现？
A:You can also use beginUpdates method followed by the endUpdates method to animate the change in the row heights without reloading the cell.

###27.什么是 Method Swizzle，什么情况下会使用？
A:Method Swizzle 是一种改变已存在方法实现的技术。当我们有改变已存在方法实现的需求是使用。

###28.为什么当 Core Animation 完成时，layer 又会恢复到原先的状态？
A: Layer 有两个独立的图层: 1.modelLayer;2.presentationLayer. 动画是在 presentationLayer 上，所以当动画完成时， layer 又恢复到原先的状态。

###29.你会如何存储用户的一些敏感信息，如登录的 token。
A:KeyChain services

###30.什么时候会使用 Core Graphics，有什么注意事项么？
A:当需要绘制的内容不能用 UIKit 实现时，会使用 Core Graphics。

注意事项：

	* 坐标系统
	* 线条的宽度尽量使用整数
	* 缓存代价昂贵的数据

###31.使用 Block 时需要注意哪些问题？
A:

* 循环引用
* 对象的生命周期会延长


###32.performSelector:withObject:afterDelay: 内部大概是怎么实现的，有什么注意事项么？


###33.如何播放 GIF 图片，有什么优化方案么？
A: 用 ImageIO 框架为每一帧动画创建一个 UIImage 对象，把这些对象聚合成一个 animatedImages 赋给 UIImageView，然后根据 GIF 的文件信息设置好 animationDuration 和 animationRepeatCount，最后调用 startAnimating。

优化方案：
	
	* variable frame delays -- One approach to support variable frame delays with UIImageView is to find the greatest common divisor of all frame delays and slot longer frames multiple times in a row into the animatedImages array.

	* memory implications -- Whenever memory is the constraint, instead of storing the solution to a problem one has to recalculate it. In our case we needed a way to load and decode the frames just in time before they were displayed, and to purge the ones that were no longer on screen.

Reference:

	* [How FlipBoard Play Animated GIFs On iOS](http://engineering.flipboard.com/2014/05/animated-gif/)


###34.有哪几种方式可以对图片进行缩放，使用 CoreGraphics 缩放时有什么注意事项？
A:

* UIGraphicsBeginImageContextWithOptions & UIImage -drawInRect:(UIKit)
* CGBitmapContextCreate & CGContextDrawImage(Core Graphics)
* CGImageSourceCreateThumbnailAtIndex(Image IO)
* Lanczos Resampling with Core Image(Core Image)
* vImage in Accelerate(vImage)

###35.哪些途径可以让 ViewController 瘦下来？
A: 我们的 App 基本都是使用类 MVC 架构，要想让 ViewController 瘦下来，基本的想法肯定是把和 ViewController 弱相关的代码移出去，移到哪？要么是称多 MV 中， 要么移到一个辅助对象中。所以基本做法应该是把弱业务代码移到 Model 中，和 View 相关的代码移到 View 中；甚至引入一个辅助对象来分担可以分离出去的职责。

Reference: [更轻量的 View Controllers](https://objccn.io/issue-1-1/)  

###36.有哪些常见的 Crash 场景？
A:

* 数组越界
* 向数组或字典中插入的对象为 nil
* 向已经销毁的对象发送消息
* 内存管理出错
* 调用一个对象不支持的方法
* 看门狗超时

###37.设计一套大文件（如上百M的视频）下载方案。

###38.如果让你来实现 dispatch_once，你会怎么做？
A:

```
void SpinlockOnce(dispatch_once_t *predicate, dispatch_block_t block) {
        static OSSpinLock lock = OS_SPINLOCK_INIT;

        OSSpinLockLock(&lock);
        if(!*predicate) {
            block();
            *predicate = 1;
        }
        OSSpinLockUnlock(&lock);
    }
```

Reference:
[Friday Q&A 2014-06-06: Secrets of dispatch_once](https://www.mikeash.com/pyblog/friday-qa-2014-06-06-secrets-of-dispatch_once.html)


Reference:https://github.com/lzyy/iOS-Developer-Interview-Questions

