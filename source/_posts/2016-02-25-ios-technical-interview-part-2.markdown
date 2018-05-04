---
layout: post
title: "iOS 面试题汇总(二)"
date: 2016-02-25 16:54:55 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS, Technical, Interview,
description: iOS Technical Interview questions about property attributes, weak, copy, arc and so on.
---
###1. @property中有哪些属性关键字？/ @property 后面可以有哪些修饰符？  
A:
	
* Write Serialization(not general thread safety): nonatomic, atomic (default)
* Mutability: readonly,readwrite (default)
* API Contron: getter = methodname, setter = mehtodname:
* Memory Management: copy, strong (default), weak, unsafe_unretained, assign

###2. ARC下，不显式指定任何属性关键字时，默认的关键字都有哪些？  
A: 
	
* Write Serialization: atomic
* Mutability: readwrite
* Memory Management: assign(non-object) strong(object)
	
###3. 什么情况使用 weak 关键字，相比 assign 有什么不同？  
A:
	什么情况使用 weak 关键字？
	
* 强引用关系的对象会发生循环引用时用weak关键字；
* 已经被强引用的对象，如Interface Builder创建的视图被声明为weak。
	
weak 相比 assign 有什么不同？
	
* weak 必须修饰对象，assign还可以修饰非对象；
* weak 修饰的属性所指向的对象销毁后，该属性会被置为nil，而assign修改的属性所指向的对象销毁后，属性仍然指向对象分配的内存地址成为野指针。
<!-- more -->

###4. weak属性需要在dealloc中置nil么？  
A: 不需要，因为 weak 属性不影响对象的引用计数。

###5. 用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？

* 对非集合类对象的copy操作
* 集合类对象的copy与mutableCopy
	
A:
为什么？
因为父类指针可以指向子类对象,而使用 copy 可以让本对象的属性不受外界影响,无论给我传入是一个可变对象还是不可对象,我本身持有的就是一个不可变的副本.
	
如果改用strong关键字，可能造成什么问题？
如果我们使用是 strong ,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么会影响该属性.
	
非集合类对象的copy与mutableCopy：
对 immutable 对象进行 copy 操作，是指针复制，mutableCopy 操作时内容复制；对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。用代码简单表示如下：
	
```objc
[immutableObject copy] // 浅复制
[immutableObject mutableCopy] //深复制
[mutableObject copy] //深复制
[mutableObject mutableCopy] //深复制
```
	
实例验证：
	
```objc
NSMutableString *string = [NSMutableString stringWithString:@"origin"];//copy
NSString *stringCopy = [string copy];
	
// 查看内存，会发现 string、stringCopy 内存地址都不一样，说明此时都是做内容拷贝、深拷贝。即使你进行如下操作：
[string appendString:@"origion!"]
	
// stringCopy 的值也不会因此改变，但是如果不使用 copy，stringCopy 的值就会被改变。
```
	
集合类对象的copy与mutableCopy:
对 immutable 对象进行 copy，是指针复制， mutableCopy 是内容复制；对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。但是：集合对象的内容复制仅限于对象本身，对象元素仍然是指针复制。用代码简单表示如下：
	
```objc
[immutableObject copy] // 浅复制
[immutableObject mutableCopy] //单层深复制
[mutableObject copy] //单层深复制
[mutableObject mutableCopy] //单层深复制
```
	
实例验证：
	
```objc
// 下面先看集合类immutable对象使用 copy 和 mutableCopy 的一个例子：
NSArray *array = @[@[@"a", @"b"], @[@"c", @"d"]];
NSArray *copyArray = [array copy];
NSMutableArray *mCopyArray = [array mutableCopy];
// 查看内容，可以看到 copyArray 和 array 的地址是一样的，而 mCopyArray 和 array 的地址	是不同的。说明 copy 操作进行了指针拷贝，mutableCopy 进行了内容拷贝。但需要强调的是：此处的内	容拷贝，仅仅是拷贝 array 这个对象，array 集合内部的元素仍然是指针拷贝。这和上面的非集合 	immutable 对象的拷贝还是挺相似的，那么mutable对象的拷贝会不会类似呢？我们继续往下，看 	mutable 对象拷贝的例子：
	
NSMutableArray *array = [NSMutableArray arrayWithObjects:[NSMutableString 	stringWithString:@"a"],@"b",@"c",nil];
NSArray *copyArray = [array copy];
NSMutableArray *mCopyArray = [array mutableCopy];

// 查看内存，如我们所料，copyArray、mCopyArray和 array 的内存地址都不一样，说明 	copyArray、mCopyArray 都对 array 进行了内容拷贝。
	
```
	
###6. 这个写法会出什么问题： @property (copy) NSMutableArray *array?  
A:

* 问题一：由于属性声明为 copy, 所以它最终指向的是一个 NSArray，对它调用 NSMutableArray 中定义的方法会导致应用奔溃；  
* 问题二：如果是开发 iOS 程序，这会影响性能。

###7. 怎么用 copy 关键字？  
A:声明属性时用 copy 做它的内存管理 attribute，例如: `@property (nonatomic, copy) NSString *name;`. 

###8. 如何让自己的类用 copy 修饰符？如何重写带 copy 关键字的 setter？  
A:

> Any object that you wish to set for a copy property must support NSCopying, which means that it should conform to the NSCopying protocol.
> Only classes that define an “immutable vs. mutable” distinction should adopt this protocol.   

如何让自己的类用 copy 修饰符？  

让类遵守 NSCopying 协议便可使用 copy 修饰符。  

如何重写带 copy 关键字的 setter？  

在 setter 中调用 copy 方法的结果赋值。  

###9. protocol 和 category 中如何使用 @property?  
A:
	
* protocol 像类一样声明 property, 遵守协议的类需要合成实例变量或实现对应的 getter 和 setter；
* category 本身不支持添加实例变量，所以我们为它声明的属性应该能通过组合已有属性来实现对应的 getter 和 setter；如果声明的属性需要引入新的实例变量则要通过关联引用来辅助实现对应的 getter 和 setter。

###10. @property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的?  
A:@property 的本质是什么？
	@property = ivar + getter + setter;
	
ivar、getter、setter 是如何生成并添加到这个类中的?  
“自动合成”( autosynthesis)
完成属性定义后，编译器会自动编写访问这些属性所需的方法，此过程叫做“自动合成”(autosynthesis)。需要强调的是，这个过程由编译器在编译期执行，所以编辑器里看不到这些“合成方法”(synthesized method)的源代码。除了生成方法代码 getter、setter 之外，编译器还要自动向类中添加适当类型的实例变量，并且在属性名前面加下划线，以此作为实例变量的名字。

###11. @synthesize和@dynamic分别有什么作用？  
A: 

* @synthesize 有两个作用：一是指定用来合成属性的后备实例变量；二是为协议中声明的属性生成存取方法；
* @dynamic 告诉编译器属性的 setter 与 getter 方法由用户自己实现，不自动合成。

###12. 在有了自动合成属性实例变量之后，@synthesize还有哪些使用场景？  
A:

* 自定义自动合成的实例变量的名称；
* 为协议中声明的属性合成 getter 和 setter 方法。

###13. @synthesize合成实例变量的规则是什么？假如property名为foo，存在一个名为_foo的实例变量，那么还会自动合成新变量么？  
A:
@synthesize合成实例变量的规则是什么？
假设声明了propertyname的属性，默认会合成_propertyname的实例变量，也可以自定义合成实例变量的名称，如果目标合成实例变量已经存在则不再合成。

假如property名为foo，存在一个名为_foo的实例变量，那么还会自动合成新变量么？
不会。
	
这里还可以引伸，我们知道默认情况下编译器会为自动合成实例变量，那么当开发者也参与到存取方法的实现中来时，自动合成实例变量又会遵循怎样的规则？  
	
* 如果声明的是读写属性，开发者只实现了 getter 或 setter，那么编译器会合成实例变量，两者都实现时则不合成实例变量；
* 如果是声明的是只读属性，开发者没有 getter，那么编译器会合成实例变量，否则不会合成。
	
Reference:[iOS automatic @synthesize without creating an ivar](https://stackoverflow.com/questions/12933785/ios-automatic-synthesize-without-creating-an-ivar)  
	
###14. ARC通过什么方式帮助开发者管理内存？  
A:

> ARC works by adding code at compile time to ensure that objects live as long as necessary, but no longer.

###15. objc使用什么机制管理对象内存？  
A: 引用计数机制。

###16. 不手动指定autoreleasepool的前提下，一个autorealese对象在什么时刻释放？（比如在一个vc的viewDidLoad中创建）  
A: Autorelease对象出了作用域之后，会被添加到最近一次创建的自动释放池中，并会在当前的 runloop 迭代结束时释放。~~如果在一个vc的viewDidLoad中创建一个 Autorelease对象，那么该对象会在 viewDidAppear 方法执行前就被销毁了。~~

###17. 苹果是如何实现autoreleasepool的?  
A: autoreleasepool 正如它的名字，它像一个池子，所以的自动释放对象都被添加到这个池子，当池子要排干时，里面所有的对象的引用计数都会减一，计数为零的对象就被销毁了，苹果就是大致按这种原理来实现 autoreleasepool的。

###18. 使用block时什么情况会发生引用循环，如何解决？  
A:一个对象中强引用了block，在block中又使用了该对象，就会发生循环引用。
解决方法是将该对象使用`__weak`修饰符修饰之后再在block中使用。

```
id __weak weakSelf = self，或者 __weak __typeof(self)weakSelf = self
```

有人可能会说用 `__block` 修饰符也可以，但我不认为这是正确的做法，`__block` 强调的是对变量存储类型，而不是对象的引用计数，最后是通过将对象置为 nil 来打破引用循环，这本质是手动打破引用循环，`__block` 修饰符侧重点是变量存储类型，它让我们在 block 内能修改变量。

这里便自然可以引申出探讨 `__weak` 和 `__block` 的区别。

> You can specify that an imported variable be mutable—that is, read-write— by applying the `__block` storage type modifier. `__block` storage is similar to, but mutually exclusive of, the register, auto, and static storage types for local variables.
> `__block` variables live in storage that is shared between the lexical scope of the variable and all blocks and block copies declared or created within the variable’s lexical scope. Thus, the storage will survive the destruction of the stack frame if any copies of the blocks declared within the frame survive beyond the end of the frame (for example, by being enqueued somewhere for later execution). Multiple blocks in a given lexical scope can simultaneously use a shared variable.


> `__weak` specifies a reference that does not keep the referenced object alive. A weak reference is set to nil when there are no strong references to the object.

还可以引申出 `__weak` 是怎么实现的。

`__weak` 是通过 weak 表来实现的，它类似 hash 表，weak 变量的地址为键，引用的对象为值，当引用的对象销毁时，通过用对象的地址在weak表中反向查到对应的变量，并将其置为 nil,然后从表中删除。

Reference:[What is the difference between a __weak and a __block reference?](https://stackoverflow.com/questions/11773342/what-is-the-difference-between-a-weak-and-a-block-reference)  
[Variable Qualifiers](https://developer.apple.com/library/content/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)  
[The __block Storage Type](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Blocks/Articles/bxVariables.html#//apple_ref/doc/uid/TP40007502-CH6-SW1)  
[How does the clang implement the weak ptr?](https://stackoverflow.com/questions/40946970/how-does-the-clang-implement-the-weak-ptr)  

###19. 在block内如何修改block外部变量？  
A:使用 _block 存储类型修饰外部变量。

###20. 使用系统的某些block api（如UIView的block版本写动画时），是否也考虑引用循环问题?   
A:使用系统的block api要不要考虑引用循环问题取决于它会不会造成引用循环，如果不会造成引用循环则不需要考虑，反之则需要考虑。UIView的block版本写动画时不需要考虑引用循环问题。

###21. IBOutlet连出来的视图属性为什么可以被设置成weak?  
A:IBOutlet 连出来的视图是视图层级的组成部分，对它已经有一个强引用，所以它可以被设置为 weak。

###22. IB中User Defined Runtime Attributes如何使用？  
A:
> Add initialization of custom runtime attributes to objects that do not have a corresponding Interface Builder inspector.

###23. 若一个类有实例变量 NSString *_foo ，调用setValue:forKey:时，可以以foo还是 _foo 作为key？  
A:都可以。

###24. KVC的keyPath中的集合运算符如何使用？  
A: 

* 必须是集合对象
* KVC支持的集合运算符
* 正确的  Operator key path 格式

###25. KVC和KVO的keyPath一定是属性么？  
A:KVO支持实例变量。

###26. `addObserver:forKeyPath:options:context:`各个参数的作用分别是什么，observer中需要实现哪个方法才能获得KVO回调？  
A:
	
| Parameter | Function |
| --------- | -------- |
| anObserver | The object to register for KVO notifications. The observer must implement the key-value observing method observeValueForKeyPath:ofObject:change:context:. |
| keyPath | The key path, relative to the receiver, of the property to observe. This value must not be nil. |
| options | A combination of the NSKeyValueObservingOptions values that specifies what is included in observation notifications. For possible values, see NSKeyValueObservingOptions. |
| context | Arbitrary data that is passed to anObserver in observeValueForKeyPath:ofObject:change:context:. |
	
observer中需要实现`- observeValueForKeyPath:ofObject:change:context:`才能获得KVO回调。

###27. 如何手动触发一个value的KVO?  
A:改变值之前调用 `willChangeValueForKey:`，改变值之后调用 `didChangeValueForKey:`。
	

###28. 如何关闭默认的KVO的默认实现，并进入自定义的KVO实现？  
A: 

* A class that implements manual notification must override the NSObject implementation of `automaticallyNotifiesObserversForKey:`.For properties that perform manual notification, the subclass implementation of `automaticallyNotifiesObserversForKey:` should return NO. 
* To implement manual observer notification, you invoke `willChangeValueForKey:` before changing the value, and `didChangeValueForKey:` after changing the value. 

###29. apple用什么方式实现对一个对象的KVO？  
A:
> Automatic key-value observing is implemented using a technique called isa-swizzling.

###30. GCD的队列（dispatch_queue_t）分哪两种类型？  
A: 
	
* DISPATCH_QUEUE_SERIAL
* DISPATCH_QUEUE_CONCURRENT

###31. 如何用GCD同步若干个异步调用？（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）  
A:
> Dispatch groups are a way to block a thread until one or more tasks finish executing. 
	
```
		dispatch_queue_t queue = 		dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
		dispatch_group_t group = dispatch_group_create();
		dispatch_group_async(group, queue, ^{ /*加载图片1 */ });
		dispatch_group_async(group, queue, ^{ /*加载图片2 */ });
		dispatch_group_async(group, queue, ^{ /*加载图片3 */ }); 
		dispatch_group_notify(group, dispatch_get_main_queue(), ^{
		        // 合并图片
		});
```
	
###32. dispatch_barrier_async的作用是什么？  
A:
> Submits a barrier block for asynchronous execution and returns immediately.
> A dispatch barrier allows you to create a synchronization point within a concurrent dispatch queue. When it encounters a barrier, a concurrent queue delays the execution of the barrier block (or any further blocks) until all blocks submitted before the barrier finish executing. At that point, the barrier block executes by itself. Upon completion, the queue resumes its normal execution behavior.

###33. 苹果为什么要废弃dispatch_get_current_queue？  
A:dispatch_get_current_queue 容易造成死锁。

###34. 以下代码运行结果如何？

```
- (void)viewDidLoad
{
	[super viewDidLoad];
	NSLog(@"1");
	dispatch_sync(dispatch_get_main_queue(), ^{
	    NSLog(@"2");
	});
	NSLog(@"3");
}
```

A: 只输出：1 。发生主线程锁死。

###35. runloop和线程有什么关系？  
A:
>Run loops are intended for situations where you want more interactivity with the thread. 
	
当我们和线程有很多交互时，可以为线程配置一个 RunLoop.
	
```
	- (void)networkRunLoopThreadEntry
    // This thread runs all of our network operation run loop callbacks.
	{
    assert( ! [NSThread isMainThread] );
    while (YES) {
        NSAutoreleasePool * pool;

        pool = [[NSAutoreleasePool alloc] init];
        assert(pool != nil);

        [[NSRunLoop currentRunLoop] run];

        [pool drain];
    }
    assert(NO);
	}
```

###36. runloop的mode作用是什么？  
A:
> A run loop mode is a collection of input sources and timers to be monitored and a collection of run loop observers to be notified.
>Each time you run your run loop, you specify (either explicitly or implicitly) a particular “mode” in which to run. During that pass of the run loop, only sources associated with that mode are monitored and allowed to deliver their events. (Similarly, only observers associated with that mode are notified of the run loop’s progress.) Sources associated with other modes hold on to any new events until subsequent passes through the loop in the appropriate mode.

mode的作用是设置好需要监控的 input sources 和 timers，以及需要通知的 run loop observers。

###37. 以+ scheduledTimerWithTimeInterval...的方式触发的timer，在滑动页面上的列表时，timer会暂定回调，为什么？如何解决？  
A:滑动页面的列表时 RunLoop 会从 NSDefaultRunLoopMode 切换到 NSEventTrackingRunLoopMode，而以+ scheduledTimerWithTimeInterval...的方式触发的timer默认是在 NSDefaultRunLoopMode 下被监控，所以会出现暂停回调。

如何解决？
解决办法自然是希望 timer 触发的事件不被限制，也就是说在 NSEventTrackingRunLoopMode 下也能被监控，可以通过把 timer 加入到 NSRunLoopCommonModes 中，因为关联到它的 timer 也会被相应关联到 NSEventTrackingRunLoopMode 中。

> NSRunLoopCommonModes (Cocoa)
> kCFRunLoopCommonModes (Core Foundation)
> This is a configurable group of commonly used modes. Associating an input source with this mode also associates it with each of the modes in the group. For Cocoa applications, this set includes the default, modal, and event tracking modes by default. Core Foundation includes just the default mode initially. You can add custom modes to the set using the CFRunLoopAddCommonMode function.

```
//将timer添加到NSDefaultRunLoopMode中
[NSTimer scheduledTimerWithTimeInterval:1.0
     target:self
     selector:@selector(timerTick:)
     userInfo:nil
     repeats:YES];
//然后再添加到NSRunLoopCommonModes里
NSTimer *timer = [NSTimer timerWithTimeInterval:1.0
     target:self
     selector:@selector(timerTick:)
     userInfo:nil
     repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

###38. 猜想runloop内部是如何实现的？  
A:
> It is a loop your thread enters and uses to run event handlers in response to incoming events.


###39. runtime 如何实现 weak 属性?/runtime如何实现weak变量的自动置nil？  
A: runtime 将 weak 属性的后备变量作为键，引用的对象的地址作为值存入到 weak 表中，它类似 hash 表，当引用的对象销毁时，在 weak 表中用此地址反向查找对应的变量，将其置为 nil, 并将其从 weak 表中删除。

###40. objc中向一个对象发送消息[obj foo]和objc_msgSend()函数之间有什么关系？  
A:
> In Objective-C, messages aren’t bound to method implementations until runtime. 
> The compiler converts a message expression,

> [receiver message]
> into a call on a messaging function, objc_msgSend. 

编译器会把向一个对象发送的消息转化成一次objc_msgSend()函数调用。

###41. objc中向一个nil对象发送消息将会发生什么？  
A:

* 如果一个方法返回值是一个对象，那么发送给nil的消息将返回nil;
* 如果方法返回值为指针类型，其指针大小为小于或者等于sizeof(void*)，float，double，long double 或者 long long 的整型标量，发送给 nil 的消息将返回0;
* 如果方法返回值为结构体,发送给 nil 的消息将返回0。结构体中各个字段的值将都是0
* 如果方法的返回值不是上述提到的几种情况，那么发送给 nil 的消息的返回值将是未定义的

###42. _objc_msgForward 函数是做什么的，直接调用它将会发生什么？  
A:当向一个对象发送一条消息，但它并没有实现的时候，_objc_msgForward会尝试做消息转发。直接调用_objc_msgForward是非常危险的事，如果用不好会直接导致程序Crash，但是如果用得好，能做很多非常酷的事。

###43. 能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？  
A:

* 不能向编译后得到的类中增加实例变量；
* 能向运行时创建的类中添加实例变量；

解释:

* 因为编译后的类已经注册在 runtime 中，类结构体中的 objc_ivar_list 实例变量的链表 和 instance_size 实例变量的内存大小已经确定，同时runtime 会调用 class_setIvarLayout 或 class_setWeakIvarLayout 来处理 strong weak 引用。所以不能向存在的类中添加实例变量；
* 运行时创建的类是可以添加实例变量，调用 class_addIvar 函数。但是得在调用 objc_allocateClassPair 之后，objc_registerClassPair 之前，原因同上。

###44. 一个objc对象如何进行内存布局？（考虑有父类的情况）  
A:
Objective-C 对象的结构图  
ISA指针  
根类的实例变量  
倒数第二层父类的实例变量  
...  
父类的实例变量  
类的实例变量  

###45. 一个objc对象的isa的指针指向什么？有什么作用？  
A:
> The isa pointer, as the name suggests, points to the object's class which maintains a dispatch table.

指向对象的类。

###46. 下面的代码输出什么？ 

```
@implementation Son : Father
- (id)init
{
    self = [super init];
    if (self) {
        NSLog(@"%@", NSStringFromClass([self class]));
        NSLog(@"%@", NSStringFromClass([super class]));
    }
    return self;
}
@end
```

A:都输出 Son.
super 是一个 Magic Keyword， 它本质是一个编译器标示符，和 self 是指向的同一个消息接受者！他们两个的不同点在于：super 会告诉编译器，调用 class 这个方法时，要去父类的方法，而不是本类里的。

###47. 什么时候会报unrecognized selector的异常？  
A:当调用该对象上某个方法,而该对象上没有实现这个方法的时候。

###48. BAD_ACCESS在什么情况下出现？  
A:

* 访问了野指针，比如对一个已经释放的对象执行了release、访问已经释放对象的成员变量或者发消息。
* 死循环

###49. 如何调试BAD_ACCESS错误?  
A:

* Enable Zombie Objects
* Enable Address Sanitizer
* 重写object的respondsToSelector方法，显示出现EXEC_BAD_ACCESS前访问的最后一个object

###50. lldb（gdb）常用的调试命令？  
A: 

```
lldb po
lldb p
lldb expression
lldb bt
lldb c
```

###51. 使用runtime Associate方法关联的对象，需要在主对象dealloc的时候释放么？  
A:不需要。

###52. runtime如何通过selector找到对应的IMP地址？（分别考虑类方法和实例方法）  
A:每一个类对象或对象中有一个方法列表,方法列表中记录着方法的名称,方法实现,以及参数类型,其实selector本质就是方法名称,通过这个方法名称就可以在方法列表中找到对应的方法实现.

###53. objc中的类方法和实例方法有什么本质区别和联系？  
A:
类方法： 

类方法是属于类对象的  
类方法只能通过类对象调用  
类方法中的self是类对象  
类方法可以调用其他的类方法  
类方法中不能访问成员变量  
类方法中不定直接调用对象方法  

实例方法：  

实例方法是属于实例对象的  
实例方法只能通过实例对象调用  
实例方法中的self是实例对象  
实例方法中可以访问成员变量  
实例方法中直接调用实例方法  
实例方法中也可以调用类方法(通过类名)  

扩展阅读：  
o [上级向的十个iOS面试问题](http://onevcat.com/2013/04/ios-interview/)  
o [百度面试](http://studentdeng.github.io/blog/2014/02/11/baidu-interview/)  

Reference:  
o [iOS Interview](http://blog.sunnyxx.com/2015/07/04/ios-interview/)  
o [iOS Interview Questions](https://github.com/ChenYilong/iOSInterviewQuestions)


