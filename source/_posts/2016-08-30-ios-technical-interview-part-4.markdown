---
layout: post
title: "iOS 面试题汇总(四)"
date: 2016-08-30 10:43:36 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS Technical Interview
description: iOS Technical Interview  
---

###1.熟悉 CocoaPods 么？能大概讲一下工作原理么？
A:熟悉，CocoaPods 是一个依赖管理工具，它通过 Podfile 来表达依赖，每个依赖都有一个 podspec 。 podspec 文件存储着该依赖的基本信息：包含哪些文件，静态库，资源文件等，依赖哪些其他第三方库、系统框架，之后 pod 会创建一个工程，把这个库以 target 的形式包含进来，应用则依赖这个 pod 工程。

###2.最常用的版本控制工具是什么，能大概讲讲原理么？
A: 最常用的版本控制工具是 Git。它的大致原理是工程每次提交时所有更改的文件都会生成快照，快照代表当前版本的文件，这些快照通过树型结构组织起来，树随着工程更改生长，通过这种方式来进行版本管理。

###3.你一般是怎么用 Instruments 的？
A:根据想要分析的问题，选择对应的 instruments 来分析应用的行为，有疑问的地方参考 Instruments User Guide．

###4.你一般是如何调试 Bug 的？
A:对于容易复现的 bug，根据复现的步骤来调试代码会比较容易定位问题，之后解决问题；对于不容易复现的 bug, 调试就比较难了，配合源码，猜测问题可能出现的原因，一步步验证假设来定位 bug，确定 bug 产生的原因才好解决。

###5.你在你的项目中用到了哪些设计模式？
A: 

创建型：
	
* 工厂方法(Factory Method)
* 原型(Prototype)
* 单例(Singleton)

结构型：

* 适配器(Adapter)
* 桥接(Bridge)
* 组成(Composite)
* 装饰(Decorator)
* 外观(Facade)
* 享元(Flyweight)

行为型：

* 职责链（Chain Of Responsibility）
* 命令(Command)
* 解释器(Interpreter)
* 迭代器(Iterator)
* 中介者(Mediator)
* 备忘录(Memento)
* 观察者(Observer)
* 状态(State)
* 策略(Strategy)
* 模板方法(Template Method)

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

1. 太多的状态，难以理解、调试、测试；
2. 它自己控制自己的创建和生命周期违反了单一原则；
3. 由于它的全局变量属性容易导致代码高藕合；

###7.如何把一个包含自定义对象的数组序列化到磁盘？
A:自定义对象需要实现 NSCoding 协议，然后可以调用 NSKeyedArchiver 的`+ (BOOL)archiveRootObject:(id)rootObject toFile:(NSString *)path`方法将它序列化到磁盘。

###8.iOS 的沙盒目录结构是怎样的？ App Bundle 里面都有什么？
A:

iOS 的沙盒目录结构如下:

AppName.app  
Documents/  
Library/  
tmp/  

App Bundle 里面有 Info.plist, Executeable, Resource files and Other support files.

Reference:  

* File System Programming Guide  

###9.iOS 7的多任务添加了哪两个新的 API? 各自的使用场景是什么？
A:

* Apps that regularly update their content by contacting a server can register with the system and be launched periodically to retrieve that content in the background. To register, include the UIBackgroundModes key with the fetch value in your app’s Info.plist file. Then, when your app is launched, call the `setMinimumBackgroundFetchInterval:` method to determine how often it receives update messages. Finally, you must also implement the `application:performFetchWithCompletionHandler:` method in your app delegate.
* Apps that use push notifications to notify the user that new content is available can fetch the content in the background. To support this mode, include the UIBackgroundModes key with the remote-notification value in your app’s Info.plist file. You must also implement the `application:didReceiveRemoteNotification:fetchCompletionHandler:` method in your app delegate.

Reference: What's New in iOS

###10.Objective-C 的 class 是如何实现的？Selector 是如何被转化为 C 语言的函数调用的？
A:
Objective-C 的 class 是如何实现的？

Objective-C 的 class 是通过 C 语言的结构体来实现的。编译器会将类按照根类实例变量，倒数第二根类实例变量，以此类推到父类实例变量，类实例变量的内存布局转换成结构体。

Selector 是如何被转化为 C 语言的函数调用的？  
在 Objective-C 中，消息在 runtime 时才会绑定到方法实现上。编译器将消息表达式 `[receiver message]` 转换为对函数 `objc_msgSend` 的调用。函数接收消息接收者和消息中方法的名字即 Selector 作为它的两个主要参数，所有传入消息的参数也会一并交给 `objc_msgSend`函数。

###11.UIScrollView 大概是如何实现的，它是如何捕捉、响应手势的？
A:

UIScrollView 大概是如何实现的？  

UIScrollView 是通过巧妙利用 UIView 已有属性实现的，它通过改变 UIScrollView bounds 的 origin 来达到移到视图的效果，改变 transforms 来实现视图缩放。

它是如何捕捉、响应手势的？  

它可能有两种方法来做这件事：一是自己实现继承自 UIResponder 用来响应 touch event 的 `touchesBegan:withEvent:` 等一系列方法；二是添加 UIGestureRecognizer.  

于是想办法来验证，首先断点打印输出附加在 scrollview 的 gesture recognizer:

```
(lldb) po self.gestureRecognizers
<__NSArrayI 0x60000025bbd0>(
<UIScrollViewDelayedTouchesBeganGestureRecognizer: 0x6000001e3800; state = Failed; delaysTouchesBegan = YES; view = <InfiniteScrollView 0x7f89ce813800>; target= <(action=delayed:, target=<InfiniteScrollView 0x7f89ce813800>)>>,
<UIScrollViewPanGestureRecognizer: 0x7f89ce613480; state = Began; delaysTouchesEnded = NO; view = <InfiniteScrollView 0x7f89ce813800>; target= <(action=handlePan:, target=<InfiniteScrollView 0x7f89ce813800>)>>,
<_UIDragAutoScrollGestureRecognizer: 0x6000007a6580; state = Possible; cancelsTouchesInView = NO; delaysTouchesEnded = NO; view = <InfiniteScrollView 0x7f89ce813800>; target= <(action=_handleAutoScroll:, target=<InfiniteScrollView 0x7f89ce813800>)>>
)
```

可以看到上面确实添加了好几个手势识别，例如 UIScrollViewPanGestureRecognizer，这样可以断定它是使用添加手势的方法来捕捉和响应手势的。  

更进一步，我们可以通过置换方法的办法，来确认 scrollview 是否使用了 `touchesBegan:withEvent:` 等方法来捕捉和响应手势，具体而言可以像这样：  

```objc
    Method origin = class_getInstanceMethod(InfiniteScrollView.class, @selector(touchesBegan:withEvent:));
    Method modifiedMethod = class_getInstanceMethod(self.class, @selector(dml_touchesBegan:withEvent:));
    method_exchangeImplementations(origin, modifiedMethod);
```

经确认，scrollview 并没有使用相关方法。

###12.Objective-C 如何对已有的方法，添加自己的功能代码以实现类似记录日志这样的功能？
A: 使用 AOP, 给方法增加切面。

###13.`+load` 和 `+initialize` 的区别是什么？
A:

* load:Invoked whenever a class or category is added to the Objective-C runtime; implement this method to perform class-specific behavior upon loading.
* initialize:Initializes the class before it receives its first message.

###14.NSOperation 相比于 GCD 有哪些优势？
A:

* 可以取消操作；
* 可以添加依赖；
* 可以 KVO 属性;
* 可以添加优先级;
* 可以复用;

###15.strong / weak / unsafe_unretained 的区别？

A: 

* strong:对象的引用计数加一;  
* weak:对象的引用计数不变，对象销毁时 weak 属性自动置为nil;  
* unsafe_unretained:对象的引用计数不变，对象销毁时 unsafe_unretained 属性成为野指针;  

你可能会奇怪，有了 weak 还要 unsafe_unretained 干什么？ 原因是 weak 是 iOS 5 才可用的，所以当你要兼容 iOS 5, 或者将 iOS 5 时代之前 MRC 代码迁移到 ARC 时会用它，除此之外我们应该使用 weak.

Reference:[What is the use of unsafe_unretained attribute?](http://stackoverflow.com/questions/15968198/what-is-the-use-of-unsafe-unretained-attribute)

###16.Objective-C 中，meta-class 指的是什么？
A:类对象的类称为 meta-class.

###17.UIView 和 CALayer 之间的关系？
A: UIView 会持有至少一个 CALayer 实例，CALayer 是 UIView 的骨架，它将 UIView 的内容绘制出来并提供动画支持。

###18.`+[UIView animateWithDuration:animations:completion:]` 内部大概是如何实现的？
A:我觉得`+[UIView animateWithDuration:animations:completion:]` 内部应该是先使能 CALayer 实例的隐式动画，之后对 UIView animatable properties 的设置会传递到 CALayer 对应属性的 setter 方法，CALayer 的 setter 方法会触发对应的隐式动画，最后禁止 CALayer 实例的隐式动画。

Reference:

* [How is UIView's +animateWithDuration:animations: implemented?](https://www.quora.com/How-is-UIViews-+animateWithDuration-animations-implemented)   
* [How UIView animations with blocks work under the hood](http://stackoverflow.com/questions/15175750/how-uiview-animations-with-blocks-work-under-the-hood)

###19.什么时候会发生「隐式动画」？
A: CALayer 的实例是支持隐式动画的，所以修改 CALayer Animatable Properties 里面的属性都可以触发隐式动画。UIView 默认禁止了 CALayer 的隐式动画，在动画块中又使能了，所以和 UIView 关联的 CALayer 实例只能在动画块中修改 Animatable UIView properties 里的属性才可以触发隐式动画。

Reference:[When exactly do implicit animations take place in iOS?](http://stackoverflow.com/questions/4749343/when-exactly-do-implicit-animations-take-place-in-ios)

###20.如何把一张大图缩小为1/4大小的缩略图？
A:

* UIGraphicsBeginImageContextWithOptions & UIImage -drawInRect:(UIKit)
* CGBitmapContextCreate & CGContextDrawImage(Core Graphics)
* CGImageSourceCreateThumbnailAtIndex(Image IO)
* Lanczos Resampling with Core Image(Core Image)
* vImage in Accelerate(vImage)

Reference:[Image Resizing Techniques](http://nshipster.com/image-resizing/)

###21.Toll-Free Bridging 是什么？什么情况下会使用？
A:
> There are a number of data types in the Core Foundation framework and the Foundation framework that can be used interchangeably. Data types that can be used interchangeably are also referred to as toll-free bridged data types. This means that you can use the same data structure as the argument to a Core Foundation function call or as the receiver of an Objective-C message invocation.

Toll-Free Bridging 是指 Core Foundation framework 和 Foundation framework 的部分数据类型可以互换使用。自然是混合使用 Core Foundation framework 和 Foundation framework 时，例如 Core Graphics, Core Animation.

###22.当系统出现内存警告时会发生什么？
A:系统会杀死后台内存占用量大的应用释放内存给当前运行的应用，当前的应用也会收到内存警告的通知。

###23.什么是 Protocol，Delegate 一般是怎么用的？
A:Protocol是一份消息合约，它定义了特定场景使用的方法集合。 Delegate 一般在对象想将其一部分职责委托出去时使用，这时我们可以通过组合的方式改变一个对象的行为方式，它比继承更灵活。

###24.UIWebView 有哪些性能问题？有没有可替代的方案。
A:

* **线程阻塞问题**--当在 native 层调用 stringByEvaluatingJavaScriptFromString 方法时，可能由于 javascript 是单线程的原因，会阻塞原有 js 代码的执行。这里我们的解决办法是在 js 端用 defer 将 iframe 的插入延后执行。
* **主线程的问题**--UIWebView 的 stringByEvaluatingJavaScriptFromString 方法必须是主线程中执行，而主线程的执行时间过长就会 block UI 的更新。所以我们应该尽量让 stringByEvaluatingJavaScriptFromString 方法执行的时间短。

Reference:[关于UIWebView的总结](http://blog.devtang.com/2012/03/24/talk-about-uiwebview-and-phonegap/)

###25.为什么 NotificationCenter 要 removeObserver? 如何实现自动 remove?
A:因为如果不 removeObserver， 收到 Observer 观察的通知时 NotificationCenter 会继续调用 Observer 对应的处理方法，此时 Observer 可能已经销毁，那么 NotificationCenter 是在访问已销毁的对象，会导致应用崩溃。

手动 remove 时，我们是在 dealloc 方法里实现的，现在想要自动，很自然会想到动态往 dealloc 中插入方法调用，这可能通过 AOP 来实现，但我们可能需要引入 AOP 实现库。既然说到动，我们可以试试 Objective-C 的 runtime。和它相关的关联引用就是将两个对象的生命周期关联起来，于是通过混写 addObserver 的方法，在这个混写方法中创建一个对象和 Observer 的生命周期关联起来，然后在这个关联对象的销毁方法中调用 removeObserver，这样就实现了自动 remove,相比于 AOP 应该更简单。

Reference:[Automatic removal of NSNotificationCenter or KVO observers](http://merowing.info/2012/03/automatic-removal-of-nsnotificationcenter-or-kvo-observers/)

###26.当 TableView 的 Cell 改变时，如何让这些改变以动画的形式呈现？
A:You can also use beginUpdates method followed by the endUpdates method to animate the change in the row heights without reloading the cell.

###27.什么是 Method Swizzle，什么情况下会使用？
A:Method Swizzle 是一种替换已存在方法实现的技术。当我们有替换已存在方法实现的需求时使用。

###28.为什么当 Core Animation 完成时，layer 又会恢复到原先的状态？
A: Layer 有两个独立的图层: 1.modelLayer;2.presentationLayer. 动画是在 presentationLayer 上，正常显示的是 modelLayer，所以当动画完成时， layer 又恢复到原先的状态。

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
A:

```
- (void)eoc_performSelector:(SEL)aSelector
withObject:(id)anArgument
afterDelay:(NSTimeInterval)delay
{
    dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(delay * NSEC_PER_SEC)), dispatch_get_global_queue(QOS_CLASS_UTILITY, 0), ^{
            typedef void (*send_type)(id, SEL, id);

            send_type func = (send_type)objc_msgSend;
            func(self, aSelector, anArgument);
            });
}
```

注意事项：
可能会导致内存泄露。因为不确定调用的方法，所以编译器不能插入合适的内存管理方法调用。例如newObject，copy之类的方法调用就会导致内存泄露。


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
A: 我们的 App 基本都是使用类 MVC 架构，要想让 ViewController 瘦下来，基本的想法肯定是把和 ViewController 弱相关的代码移出去，移到哪？要么是移到 M 或 V 中， 要么移到一个辅助对象中，这些就是可以让 ViewController 瘦下来的途径。

Reference: [更轻量的 View Controllers](https://objccn.io/issue-1-1/)  

###36.有哪些常见的 Crash 场景？
A:

* 数组越界
* 向已经销毁的对象发送消息
* 调用一个对象不支持的方法
* 向数组或字典中插入的对象为 nil
* 看门狗超时
* 线程死锁

###37.设计一套大文件（如上百M的视频）下载方案。
A:
[TCBlobDownload](https://github.com/thibaultcha/TCBlobDownload)

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


Reference:[iOS-Developer-Interview-Questions](https://github.com/lzyy/iOS-Developer-Interview-Questions)  

