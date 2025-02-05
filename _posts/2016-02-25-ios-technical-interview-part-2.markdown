---
layout: post
title: "iOS 面试题汇总(二)"
date: 2016-02-25 16:54:55 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS, Technical, Interview,
description: iOS Technical Interview questions about property attributes, weak, copy, arc and so on.
---

###1.什么情况使用 weak 关键字，相比 assign 有什么不同？  
 A:什么情况使用 weak 关键字？
	
* 强引用关系的对象会发生循环引用时用weak关键字；
* 已经被强引用的对象，如Interface Builder创建的视图被声明为weak。
	
weak 相比 assign 有什么不同？
	
* weak 必须修饰对象，assign还可以修饰非对象；
* weak 修饰的属性所指向的对象销毁后，该属性会被置为nil，而assign修改的属性所指向的对象销毁后，属性仍然指向对象分配的内存地址成为野指针。

###2.怎么用 copy 关键字？(我觉得题目应该改为什么情况使用 copy 关键字，怎么用 copy 关键字这个问法有点奇怪，比如怎么用手机，那问题的答案就是手机的使用说明书，按这样来回答问题，那我们就应该说，声明属性时加上 copy 关键字)  
 A:属性想拥有独立的赋值副本，且赋值对象的类遵守 NSCopying 协议，这时我们可以使用 copy 关键字。   

###3.这个写法会出什么问题： `@property (copy) NSMutableArray *array;`  
 A:

* 问题一：由于属性声明为 copy, 所以它最终指向的是一个 NSArray，对它调用 NSMutableArray 中定义的方法会导致应用奔溃；  
* 问题二：如果是开发 iOS 程序，这会影响性能。

扩展：

Apple 在 The Objective-C Programming Language 中介绍 atomic 时说：

> Properties are atomic by default so that synthesized accessors provide robust access to properties in a multithreaded environment—that is, the value returned from the getter or set via the setter is always fully retrieved or set regardless of what other threads are executing concurrently.
> If you specify strong, copy, or retain and do not specify nonatomic, then in a reference-counted environment, a synthesized get accessor for an object property uses a lock and retains and autoreleases the returned value—the implementation will be similar to the following:

```
[_internal lock]; // lock using an object-level lock

id result = [[value retain] autorelease];

[_internal unlock];

return result;
```

从文档的修改记录来看，最后一次修改时间是2011-10-12，维基上查了下 Mid 2011(July) 的 MacBook Air 的低配处理器参数为: 1.6 GHz (i5-2467M) dual-core Intel Core i5 with 3 MB shared L3 cacheOptional 1.8 GHz (i7-2677M) dual-core Intel Core i7 with 4 MB shared L3 cache, 内存为：2 GB (11" base model; Optional 4 GB) or 4 GB of 1333 MHz DDR3 SDRAM (all other models)，目前 Apple 还支持 iPhone 5S，它的处理器：64-bit 1.3 GHz dual-core Apple Cyclone，内存：1 GB LPDDR3 RAM，很遗憾数据好像没有全面超越，不过也比较接近了，当设备淘汰到 iPhone 6S 时，我们有理由在 iOS 开发中直接使用 atomic 关键字了。
<!--more-->
###4.如何让自己的类用 copy 修饰符？如何重写带 copy 关键字的 setter？  
 A:

> copy
> Specifies that a copy of the object should be used for assignment.
> The previous value is sent a release message.
> The copy is made by invoking the copy method. This attribute is valid only for object types, which must implement the NSCopying  protocol.

如何让自己的类用 copy 修饰符？  

让类遵守 NSCopying 协议便可使用 copy 修饰符。  

如何重写带 copy 关键字的 setter？  

在 setter 中调用 copy 方法的结果赋值。  

###5.@property 的本质是什么？ivar、getter、setter 是如何生成并添加到这个类中的  

A: 

> You typically access an object’s properties (in the sense of its attributes and relationships) through a pair of accessor (getter/setter) methods. By using accessor methods, you adhere to the principle of encapsulation (see Mechanisms Of Abstraction in Object-Oriented Programming with Objective-C). You can exercise tight control of the behavior of the getter/setter pair and the underlying state management while clients of the API remain insulated from the implementation changes.  
> 
> Although using accessor methods therefore has significant advantages, writing accessor methods is a tedious process. Moreover, aspects of the property that may be important to consumers of the API are left obscured—such as whether the accessor methods are thread-safe or whether new values are copied when set.  
> 
> Declared properties address these issues by providing the following features:  
> 
> 	•	The property declaration provides a clear, explicit specification of how the accessor methods behave.  
> 
> 	•	The compiler can synthesize accessor methods for you, according to the specification you provide in the declaration.  
> 
> 	•	Properties are represented syntactically as identifiers and are scoped, so the compiler can detect use of undeclared properties.  
> 

从 Apple 的上面的介绍，我觉得 @property 的本质是一种优雅的表达对象属性的语法。

ivar、getter、setter 是如何生成并添加到这个类中的?  

完成属性定义后，编译器会自动生成访问这些属性所需的方法，此过程叫做“自动合成”(autosynthesis)。需要强调的是，这个过程由编译器在编译期执行，所以编辑器里看不到这些“合成方法”(synthesized method)的源代码。除了生成方法代码 getter、setter 之外，编译器还要自动向类中添加适当类型的实例变量，并且在属性名前面加下划线，以此作为实例变量的名字。

###6.@protocol 和 category 中如何使用 @property  
 A:使用 @property 分为声明和实现，protocol 只需要声明，因此只需要依照 property 语法声明就好。category 不仅要声明而且要实现，因为category 本身不支持增加实例变量，所以我们需要自己实现相应的 getter 和 setter 方法，如果声明的属性不是通过组合已有属性而是引入新的实例变量，我们需要借助关联引用来实现。

###7.runtime 如何实现 weak 属性  
 A:runtime 通过 weak 表来实现 weak 属性，它类似 hash 表，以赋值对象的地址为键，weak 变量的地址为值，当赋值对象销毁时，通过对象的地址在weak表中找到对应的变量的地址，并将值其置为 nil,然后从表中删除。

###8.@property中有哪些属性关键字？/ @property 后面可以有哪些修饰符？  
 A:

* Atomicity(not general thread safety): nonatomic, atomic(default)
* Writability: readonly, readwrite(default)
* Accessor Method Names: getter = methodname, setter = methodname:
* Setter Semantics: strong, weak, copy, retain, assign(default)

###9.weak属性需要在dealloc中置nil么？  

A:不需要，因为 weak 属性不影响对象的引用计数。

###10.@synthesize和@dynamic分别有什么作用？  
 A:

> @synthesize
> You use the @synthesize directive to tell the compiler that it should synthesize the setter and/or getter methods for a property if you do not supply them within the @implementation block. The @synthesize directive also synthesizes an appropriate instance variable if it is not otherwise declared.

> @dynamic
> You use the @dynamic keyword to tell the compiler that you will fulfill the API contract implied by a property either by providing method implementations directly or at runtime using other mechanisms such as dynamic loading of code or dynamic method resolution. It suppresses the warnings that the compiler would otherwise generate if it can’t find suitable implementations. You should use it only if you know that the methods will be available at runtime.

* @synthesize 用来告诉编译器需要为属性生成相应的 setter 和 getter 方法，由于从 Xcode 4 开始支持默认自动合成，所以通常用来为协议声明的属性生成相应的 setter 和 getter 方法。也可以用来为属性指定合适的后备实例变量。
* @dynamic 用来告诉编译器你会用自己的方法来满足 API 要求，不需要它自动合成。

>Xcode 4.0 Developer Preview 4
>Objective-C: Adds default automatic synthesis of properties (iOS and 64-bit OS X). You don’t need the @synthesize directive in the implementation sections for the compiler to synthesize accessors for declared properties.

追问：我们如何通过动态方法求解(dynamic method resolution)来动态提供方法实现？  

A:Objective-C 方法是至少有 self 和 _cmd 两个参数的简单 C 函数，我们通过实现符合签名的函数，然后在 `resolveInstanceMethod:` 和 `resolveClassMethod:` 实现中通过 class_addMethod 将函数添加到类中作为方法来完成动态提供方法实现。

追问：动态方法求解(dynamic method resolution) 是如何工作的？  

追问：既然说到动态方法求解(dynamic method resolution),我们可能会联想到消息转发(message forwarding),那么动态方法求解和消息转发是什么关系？消息转发又是如何工作的呢？  

A:

> Forwarding methods (as described in Message Forwarding) and dynamic method resolution are, largely, orthogonal. A class has the opportunity to dynamically resolve a method before the forwarding mechanism kicks in. If respondsToSelector: or instancesRespondToSelector: is invoked, the dynamic method resolver is given the opportunity to provide an IMP for the selector first. If you implement resolveInstanceMethod: but want particular selectors to actually be forwarded via the forwarding mechanism, you return NO for those selectors.

动态方法求解和消息转发是正交关系。  

追问：在使用 CoreData 时，通过我们会用 @dynamic 来实现 NSManagedObject 子类声明的属性，那么 CoreData 是如何在运行时满足 API 要求的呢？

###11.ARC下，不显式指定任何属性关键字时，默认的关键字都有哪些？  
 A:

* Write Serialization: atomic
* Mutability: readwrite
* Memory Management: assign(non-object) strong(object)

###12.用@property声明的NSString（或NSArray，NSDictionary）经常使用copy关键字，为什么？如果改用strong关键字，可能造成什么问题？  
 A:为什么？  

因为这些类都有对应的可变子类，使用 copy 可以避免可变对象的修改对赋值产生影响，无论给赋值是一个可变对象还是不可对象,属性最终都拥有一个不可变的副本.
	
如果改用strong关键字，可能造成什么问题？  

如果我们使用是 strong ,那么这个属性就有可能指向一个可变对象,如果这个可变对象在外部被修改了,那么有可能会影响该属性.

###13.对非集合类对象的copy操作  
 A:对 immutable 对象进行 copy 操作，是指针复制，mutableCopy 操作时内容复制；对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。用代码简单表示如下：
	
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
	
// 查看内存，会发现 string、stringCopy 内存地址都不一样，说明此时都是做内容拷贝、深拷贝。
// 即使你进行如下操作：
[string appendString:@"origin!"]
	
// stringCopy 的值也不会因此改变，但是如果不使用 copy，stringCopy 的值就会被改变。
```

###14.集合类对象的copy与mutableCopy  
 A:对 immutable 集合类对象进行 copy，是指针复制， mutableCopy 是内容复制；对 mutable 对象进行 copy 和 mutableCopy 都是内容复制。但是：集合对象的内容复制仅限于对象本身，对象元素仍然是指针复制。用代码简单表示如下：
	
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
// 查看内容，可以看到 copyArray 和 array 的地址是一样的，
// 而 mCopyArray 和 array 的地址是不同的。
// 说明 copy 操作进行了指针拷贝，mutableCopy 进行了内容拷贝。
// 但需要强调的是：此处的内容拷贝，仅仅是拷贝 array 这个对象，
// array 集合内部的元素仍然是指针拷贝。这和上面的非集合 immutable 对象的拷贝还是挺相似的，
// 那么 mutable对象的拷贝会不会类似呢？我们继续往下，
// 看 mutable 对象拷贝的例子：
	
NSMutableArray *array = [NSMutableArray arrayWithObjects:[NSMutableString 	stringWithString:@"a"],@"b",@"c",nil];
NSArray *copyArray = [array copy];
NSMutableArray *mCopyArray = [array mutableCopy];

// 查看内存，如我们所料，copyArray、mCopyArray和 array 的内存地址都不一样，
// 说明 	copyArray、mCopyArray 都对 array 进行了内容拷贝。
	
```

###15.@synthesize合成实例变量的规则是什么？假如property名为foo，存在一个名为_foo的实例变量，那么还会自动合成新变量么？  
 A:@synthesize合成实例变量的规则是什么？  

假设声明了propertyname的属性，默认会合成_propertyname的实例变量，也可以自定义合成实例变量的名称，如果目标合成实例变量已经存在则不再合成。

假如property名为foo，存在一个名为_foo的实例变量，那么还会自动合成新变量么？  

不会。
	
这里还可以引伸，我们知道默认情况下编译器会为自动合成实例变量，那么当开发者也参与到存取方法的实现中来时，自动合成实例变量又会遵循怎样的规则？  
	
* 如果声明的是读写属性，开发者只实现了 getter 或 setter，那么编译器会合成实例变量，两者都实现时则不合成实例变量；
* 如果是声明的是只读属性，开发者没有 getter，那么编译器会合成实例变量，否则不会合成。
	
Reference:[iOS automatic @synthesize without creating an ivar](https://stackoverflow.com/questions/12933785/ios-automatic-synthesize-without-creating-an-ivar)  

###16.在有了自动合成属性实例变量之后，@synthesize还有哪些使用场景？  
 A:

* 自定义自动合成的实例变量的名称；
* 为协议中声明的属性合成 getter 和 setter 方法。

###17.objc中向一个nil对象发送消息将会发生什么？  
 A:
> In Objective-C, it is valid to send a message to nil—it simply has no effect at runtime. There are several patterns in Cocoa that take advantage of this fact. The value returned from a message to nil may also be valid:  
> 
> 	•	If the method returns an object, then a message sent to nil returns 0 (nil). For example:  `Person *motherInLaw = [[aPerson spouse] mother];`  
>
> If the spouse object here is nil, then mother is sent to nil and the method returns nil.  
>  
> 	•	If the method returns any pointer type, any integer scalar of size less than or equal to sizeof(void*), a float, a double, a long double, or a long long, then a message sent to nil returns 0.  
> 
> 	•	If the method returns a struct, as defined by the OS X ABI Function Call Guide to be returned in registers, then a message sent to nil returns 0.0 for every field in the struct. Other struct data types will not be filled with zeros.  
> 
> 	•	If the method returns anything other than the aforementioned value types, the return value of a message sent to nil is undefined.  
> 

objc中向一个nil对象发送消息在运行时不会有什么效果。

###18.objc中向一个对象发送消息[obj foo]和objc_msgSend()函数之间有什么关系？  
 A:
> In Objective-C, messages aren’t bound to method implementations until runtime. 
> The compiler converts a message expression,

> [receiver message]
> into a call on a messaging function, objc_msgSend. 

编译器会把向一个对象发送的消息转化成一次objc_msgSend()函数调用。

###19.什么时候会报unrecognized selector的异常？  
 A:当调用该对象上某个方法,而该对象上没有实现这个方法的时候。

###20.一个objc对象如何进行内存布局？（考虑有父类的情况）  
 A:
Objective-C 对象的结构图  
ISA指针  
根类的实例变量  
倒数第二层父类的实例变量  
...  
父类的实例变量  
类的实例变量  

###21.一个objc对象的isa的指针指向什么？有什么作用？  
 A:
> The isa pointer, as the name suggests, points to the object's class which maintains a dispatch table.

指向对象的类。

有什么用？  

方法调用的底层实现信赖于它。

###22.下面的代码输出什么？  

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

// Test driver code
Son *son = [Son new];
```
A:都输出 Son.
super 是一个 Magic Keyword， 它本质是一个编译器标示符，和 self 是指向的同一个消息接受者！他们两个的不同点在于：super 会告诉编译器，调用 class 这个方法时，要去调用父类的分发表里的方法，而不是调用本类分发表里的方法。于是 self 从类的方法列表， super 从父类的方法列表沿继承链向上查找，最终在 NSObject 的方法方法列表里找到方法实现，class 方法是返回接收者的类对象，而 super 只是个标识，实际上两者的接收者都是 son 实例，所以最终的结果就是 Son。  

假设我们现在基于 Son 定义 GrandSon 类，代码如下:

```
@interface GrandSon : Son

@end

@implementation GrandSon

@end

// Test driver code
GrandSon *grandSon = [GrandSon new];
```

此时输出的是什么呢？基于上述的的解释，现在的对象是 grandSon，它的类是 GrandSon，于是输出便是 GrandSon。  

>super is simply a flag to the compiler telling it where to begin searching for the method to perform; it’s used only as the receiver of a message. But self is a variable name that can be used in any number of ways, even assigned a new value.

###23. _objc_msgForward 函数是做什么的，直接调用它将会发生什么？  

A:当向一个对象发送一条消息，但它并没有实现的时候，`_objc_msgForward`会尝试做消息转发。直接调用`_objc_msgForward`是非常危险的事，如果用不好会直接导致程序Crash，但是如果用得好，能做很多非常酷的事。

###24.runtime如何实现weak变量的自动置nil？  

A: runtime 将后备变量引用对象的地址作为键，weak 属性的后备变量的地址作为值，存入到 weak 表中，它类似 hash 表，当引用的对象销毁时，通过对象的地址在weak表中找到对应的变量地址，将其值置为 nil, 并将其从 weak 表中删除，这样便实现了 weak 属性。

###25.能否向编译后得到的类中增加实例变量？能否向运行时创建的类中添加实例变量？为什么？  

A:

* 不能向编译后得到的类中增加实例变量；
* 能向运行时创建的类中添加实例变量；

解释:

* 因为编译后的类已经注册在 runtime 中，类结构体内存大小已经确定，不能为新增的实例分配内存空间，所以不能向存在的类中添加实例变量；
* 运行时创建的类是可以添加实例变量，调用 class_addIvar 函数。但是得在调用 objc_allocateClassPair 之后，objc_registerClassPair 之前，原因同上。

###26.runloop和线程有什么关系？  

A:
>Run loops are part of the fundamental infrastructure associated with threads.
	
RunLoop是线程的一部分基础设施，当我们和线程有很多交互时，可以为线程配置一个 RunLoop.
	
```objc
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

###27.runloop的mode作用是什么？  

A:
> A run loop mode is a collection of input sources and timers to be monitored and a collection of run loop observers to be notified.
>Each time you run your run loop, you specify (either explicitly or implicitly) a particular “mode” in which to run. During that pass of the run loop, only sources associated with that mode are monitored and allowed to deliver their events. (Similarly, only observers associated with that mode are notified of the run loop’s progress.) Sources associated with other modes hold on to any new events until subsequent passes through the loop in the appropriate mode.

mode 是需要监控的 input sources 和 timers的集合，以及需要通知的 run loop observers的集合。

###28.以`+ scheduledTimerWithTimeInterval...`的方式触发的timer，在滑动页面上的列表时，timer会暂定回调，为什么？如何解决？  

A:滑动页面的列表时 RunLoop 会从 NSDefaultRunLoopMode 切换到 NSEventTrackingRunLoopMode，而以`+ scheduledTimerWithTimeInterval...`的方式触发的timer默认是在 NSDefaultRunLoopMode 下被监控，所以会出现暂停回调。

如何解决？  

解决办法自然是希望 timer 触发的事件不被限制，也就是说在 NSEventTrackingRunLoopMode 下也能被监控，可以通过把 timer 加入到 NSRunLoopCommonModes 中，因为关联到它的 timer 也会被相应关联到 NSEventTrackingRunLoopMode 中。

> NSRunLoopCommonModes (Cocoa)
> kCFRunLoopCommonModes (Core Foundation)
> This is a configurable group of commonly used modes. Associating an input source with this mode also associates it with each of the modes in the group. For Cocoa applications, this set includes the default, modal, and event tracking modes by default. Core Foundation includes just the default mode initially. You can add custom modes to the set using the CFRunLoopAddCommonMode function.

```objc
//将timer添加到NSDefaultRunLoopMode中
[NSTimer scheduledTimerWithTimeInterval:1.0
     target:self
     selector:@selector(timerTick:)
     userInfo:nil
     repeats:YES];
     
//将timer添加到NSRunLoopCommonModes中
NSTimer *timer = [NSTimer timerWithTimeInterval:1.0
     target:self
     selector:@selector(timerTick:)
     userInfo:nil
     repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];
```

###29.猜想runloop内部是如何实现的？  

A:
> It is a loop your thread enters and uses to run event handlers in response to incoming events.


###30.objc使用什么机制管理对象内存？  

A: 引用计数机制。

###31.ARC通过什么方式帮助开发者管理内存？  

A:

> ARC works by adding code at compile time to ensure that objects live as long as necessary, but no longer.

ARC 通过在编译时添加代码来保证对象仅存活足够久的方式帮助开发者管理内存。

###32.不手动指定autoreleasepool的前提下，一个autorealese对象在什么时刻释放？（比如在一个vc的viewDidLoad中创建）  

A: Autorelease对象出了作用域之后，会被添加到最近一次创建的自动释放池中，并会在当前的事件循环迭代结束时释放。  

>Cocoa always expects code to be executed within an autorelease pool block, otherwise autoreleased objects do not get released and your application leaks memory. (If you send an autorelease message outside of an autorelease pool block, Cocoa logs a suitable error message.) The AppKit and UIKit frameworks process each event-loop iteration (such as a mouse down event or a tap) within an autorelease pool block.

> In the main event loop, an application continuously routes incoming events to objects for handling and, as a result of that handling, updates its appearance and state. An event loop is simply a run loop: an event-processing loop for scheduling work and coordinating the receipt of events from various input sources attached to the run loop. Every thread has access to a run loop. In all but the main thread, the run loop must be configured and run manually by your code. In Cocoa applications, the run loop for the main thread—the main event loop—is run automatically by the application object. What distinguishes the main event loop is that its primary input source receives events from the operating system that are generated by user actions—for example, tapping a view or entering text using a keyboard.

具体到一个在 VC 的 viewDidLoad 中创建的 autorealease 对象，它会在 viewDidAppear 方法之前就释放掉。我们可以通过观察 run loop 的方法更细致的确认，代码和输出结果如下：

```
// Sample result
2019-02-02 14:38:26.625904+0800 TechnologyInterviewDemo[31535:7875199] -[AutoreleaseViewController viewWillAppear:], autorelease obj: NSMutableString stand for autorelease obj
2019-02-02 14:38:27.053430+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopAfterWaiting, autorelease obj: (null)
2019-02-02 14:38:27.053711+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeTimers, autorelease obj: (null)
2019-02-02 14:38:27.053881+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeSources, autorelease obj: (null)
2019-02-02 14:38:27.053990+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeWaiting, autorelease obj: (null)
2019-02-02 14:38:27.123273+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopAfterWaiting, autorelease obj: (null)
2019-02-02 14:38:27.123548+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeTimers, autorelease obj: (null)
2019-02-02 14:38:27.123699+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeSources, autorelease obj: (null)
2019-02-02 14:38:27.123846+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeWaiting, autorelease obj: (null)
2019-02-02 14:38:27.402676+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopAfterWaiting, autorelease obj: (null)
2019-02-02 14:38:27.402871+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeTimers, autorelease obj: (null)
2019-02-02 14:38:27.402984+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeSources, autorelease obj: (null)
2019-02-02 14:38:27.403086+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeWaiting, autorelease obj: (null)
2019-02-02 14:38:27.403275+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopAfterWaiting, autorelease obj: (null)
2019-02-02 14:38:27.414662+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeTimers, autorelease obj: (null)
2019-02-02 14:38:27.414893+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeSources, autorelease obj: (null)
2019-02-02 14:38:27.415021+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeWaiting, autorelease obj: (null)
2019-02-02 14:38:27.416351+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopAfterWaiting, autorelease obj: (null)
2019-02-02 14:38:27.416551+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeTimers, autorelease obj: (null)
2019-02-02 14:38:27.416651+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeSources, autorelease obj: (null)
2019-02-02 14:38:27.416759+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeWaiting, autorelease obj: (null)
2019-02-02 14:38:27.416907+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopAfterWaiting, autorelease obj: (null)
2019-02-02 14:38:27.426358+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeTimers, autorelease obj: (null)
2019-02-02 14:38:27.426505+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeSources, autorelease obj: (null)
2019-02-02 14:38:27.426628+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeWaiting, autorelease obj: (null)
2019-02-02 14:38:27.553645+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopAfterWaiting, autorelease obj: (null)
2019-02-02 14:38:27.553896+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeTimers, autorelease obj: (null)
2019-02-02 14:38:27.553992+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeSources, autorelease obj: (null)
2019-02-02 14:38:27.554082+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopBeforeWaiting, autorelease obj: (null)
2019-02-02 14:38:27.554261+0800 TechnologyInterviewDemo[31535:7875199] kCFRunLoopAfterWaiting, autorelease obj: (null)
2019-02-02 14:38:27.555943+0800 TechnologyInterviewDemo[31535:7875199] -[AutoreleaseViewController viewDidAppear:], autorelease obj: (null)

// Sample code
@interface AutoreleaseViewController ()
{
    CFRunLoopObserverRef    observer;
}

@end

@implementation AutoreleaseViewController

__weak id reference = nil;

void myRunLoopObserver(CFRunLoopObserverRef observer, CFRunLoopActivity activity, void *info)
{
    NSString *activityName;
    
    switch (activity) {
        case kCFRunLoopEntry:
            activityName = @"kCFRunLoopEntry";
            break;

        case kCFRunLoopBeforeTimers:
            activityName = @"kCFRunLoopBeforeTimers";
            break;

        case kCFRunLoopBeforeSources:
            activityName = @"kCFRunLoopBeforeSources";
            break;

        case kCFRunLoopBeforeWaiting:
            activityName = @"kCFRunLoopBeforeWaiting";
            break;

        case kCFRunLoopAfterWaiting:
            activityName = @"kCFRunLoopAfterWaiting";
            break;

        case kCFRunLoopExit:
            activityName = @"kCFRunLoopExit";
            break;

        case kCFRunLoopAllActivities:
            activityName = @"kCFRunLoopAllActivities";
            break;
    }

    NSLog(@"%@, autorelease obj: %@", activityName, reference);
}

- (void)dealloc
{
    if (observer) {
        CFRelease(observer);
    }
    
    NSLog(@"%s, autorelease obj: %@", __func__, reference);
}

- (instancetype)initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle *)nibBundleOrNil
{
    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
    if (self) {
        // Creating a run loop observer
        NSRunLoop *myRunLoop = [NSRunLoop currentRunLoop];

        // Create a run loop observer and attach it to the run loop.
        CFRunLoopObserverContext context = {0, (__bridge void *)(self), NULL, NULL, NULL};
        CFRunLoopObserverRef observer = CFRunLoopObserverCreate(kCFAllocatorDefault,
                                                                   kCFRunLoopAllActivities, YES, 0, &myRunLoopObserver, &context);

        if (observer) {
            CFRunLoopRef cfLoop = [myRunLoop getCFRunLoop];
            CFRunLoopAddObserver(cfLoop, observer, kCFRunLoopDefaultMode);
        }
    }
    return self;
}

- (void)viewDidLoad
{
    [super viewDidLoad];
    
    NSString *str = [NSMutableString stringWithFormat:@"NSMutableString stand for autorelease obj"];
    // str是一个autorelease对象，设置一个weak的引用来观察它
    reference = str;
}
- (void)viewWillAppear:(BOOL)animated
{
    [super viewWillAppear:animated];
    NSLog(@"%s, autorelease obj: %@", __func__, reference); // Console: sunnyxx
}
- (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
    NSLog(@"%s, autorelease obj: %@", __func__, reference); // Console: (null)
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    NSLog(@"%s, autorelease obj: %@", __func__, reference);
}

- (void)viewDidDisappear:(BOOL)animated
{
    [super viewDidDisappear:animated];
    NSLog(@"%s, autorelease obj: %@", __func__, reference);
}
```

###33.BAD_ACCESS在什么情况下出现？  

A:

* 访问了野指针，比如对一个已经释放的对象执行了release、访问已经释放对象的成员变量或者发消息。
* 死循环

###34.苹果是如何实现autoreleasepool的？  

A: 苹果是通过类似数组的数据结构来实现 autoreleasepool 的，所有的自动释放对象都被添加到这个数组中，当 autoreleasepool 要排干时，里面所有的对象的引用计数都会减一，计数为零的对象就被销毁了。

###35.使用block时什么情况会发生引用循环，如何解决？  

A:一个对象强引用了block，在block中又使用了该对象，就会发生循环引用。
解决方法是将该对象使用`__weak`修饰符修饰之后再在block中使用。

```objc
id __weak weakSelf = self;
// Or
__weak __typeof(self)weakSelf = self;
```

有人可能会说用 `__block` 修饰符也可以，但我不推荐这种做法，`__block` 强调的是变量存储类型，而不是对象的引用计数，最后是通过将对象置为 nil 来打破引用循环，这本质是手动打破引用循环，很繁琐，也很容易出错。

这里便自然可以引申出探讨 `__weak` 和 `__block` 的区别。

> You can specify that an imported variable be mutable—that is, read-write— by applying the `__block` storage type modifier. `__block` storage is similar to, but mutually exclusive of, the register, auto, and static storage types for local variables.
> `__block` variables live in storage that is shared between the lexical scope of the variable and all blocks and block copies declared or created within the variable’s lexical scope. Thus, the storage will survive the destruction of the stack frame if any copies of the blocks declared within the frame survive beyond the end of the frame (for example, by being enqueued somewhere for later execution). Multiple blocks in a given lexical scope can simultaneously use a shared variable.


> `__weak` specifies a reference that does not keep the referenced object alive. A weak reference is set to nil when there are no strong references to the object.

还可以引申出 `__weak` 是怎么实现的。

`__weak` 是通过 weak 表来实现的，它类似 hash 表，以赋值对象的地址为键，weak 变量的地址为值，当赋值对象销毁时，通过对象的地址在weak表中找到对应的变量的地址，并将值其置为 nil,然后从表中删除。

Reference:[What is the difference between a __weak and a __block reference?](https://stackoverflow.com/questions/11773342/what-is-the-difference-between-a-weak-and-a-block-reference)  
[Variable Qualifiers](https://developer.apple.com/library/content/releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html)  
[The __block Storage Type](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Blocks/Articles/bxVariables.html#//apple_ref/doc/uid/TP40007502-CH6-SW1)  
[How does the clang implement the weak ptr?](https://stackoverflow.com/questions/40946970/how-does-the-clang-implement-the-weak-ptr)  

###36.在block内如何修改block外部变量？  

A:使用 `_block` 存储类型修饰外部变量。

追问：为什么加上 `_block` 就可以修改外部变量了？这是怎么实现的？  

因为加上 `_block` 修饰外部变量以后，编译器会为该变量生成相应的对象结构体，block 内部和外部都是读写这个对象结构体，而且会插入相应的内存管理代码，这样便能实现修改 block 外部变量。而不加 `_block` 时，变量都是只读的，会直接赋给 block 结构体成员变量，block 内部是通过成员变量取值，而外部仍然是直接通过变量取值。

###37.使用系统的某些block API（如UIView的block版本写动画时），是否也考虑引用循环问题？  

A:使用系统的block api要不要考虑引用循环问题取决于它会不会造成引用循环，如果不会造成引用循环则不需要考虑，反之则需要考虑。UIView的block版本写动画时不需要考虑引用循环问题。

追问：那你能举个需要考虑引用循环的系统 block API 吗？  

```
// GCD
void dispatch_async(dispatch_queue_t queue, dispatch_block_t block);

// NSNotificationCenter
- (id <NSObject>)addObserverForName:(nullable NSNotificationName)name object:(nullable id)obj queue:(nullable NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
```

###38.GCD的队列（dispatch_queue_t）分哪两种类型？  

A: 
	
* DISPATCH_QUEUE_SERIAL
* DISPATCH_QUEUE_CONCURRENT

###39.如何用GCD同步若干个异步调用？（如根据若干个url异步加载多张图片，然后在都下载完成后合成一张整图）  

A:
> Dispatch groups are a way to block a thread until one or more tasks finish executing. 
	
```objc
	dispatch_queue_t queue = 		dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	dispatch_group_t group = dispatch_group_create();
	dispatch_group_async(group, queue, ^{ /*加载图片1 */ });
	dispatch_group_async(group, queue, ^{ /*加载图片2 */ });
	dispatch_group_async(group, queue, ^{ /*加载图片3 */ }); 
	dispatch_group_notify(group, dispatch_get_main_queue(), ^{
	        // 合并图片
	});
```

###40.dispatch_barrier_async的作用是什么？  

A:
> Submits a barrier block for asynchronous execution and returns immediately.
> A dispatch barrier allows you to create a synchronization point within a concurrent dispatch queue. When it encounters a barrier, a concurrent queue delays the execution of the barrier block (or any further blocks) until all blocks submitted before the barrier finish executing. At that point, the barrier block executes by itself. Upon completion, the queue resumes its normal execution behavior.

dispatch_barrier_async 的作用是允许你在并发队列中创建一个同步点。

###41.苹果为什么要废弃dispatch_get_current_queue？  

A:dispatch_get_current_queue 容易误用造成死锁，所以苹果在iOS6废弃了dispatch_get_current_queue()方法。

例如下面示例代码：

```

- (void)deadLockFunc
{
    dispatch_queue_t queueA = dispatch_queue_create("com.yiyaaixuexi.queueA", NULL);
    dispatch_queue_t queueB = dispatch_queue_create("com.yiyaaixuexi.queueB", NULL);
    dispatch_sync(queueA, ^{
        dispatch_sync(queueB, ^{
            dispatch_block_t block = ^{
                //do something
            };
            func(queueA, block);
        });
    });
}

void func(dispatch_queue_t queue, dispatch_block_t block)
{
    if (dispatch_get_current_queue() == queue) {
        block();
    }else{
        dispatch_sync(queue, block);
    }
}
```

Reference:[被废弃的dispatch_get_current_queue](https://blog.csdn.net/yiyaaixuexi/article/details/17752925)  

###42.以下代码运行结果如何？  

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

###43.`addObserver:forKeyPath:options:context:`各个参数的作用分别是什么，observer中需要实现哪个方法才能获得KVO回调？  

A:
	
| Parameter | Function |
| --------- | :-------- |
| anObserver | The object to register for KVO notifications. The observer must implement the key-value observing method observeValueForKeyPath:ofObject:change:context:. |
| keyPath | The key path, relative to the receiver, of the property to observe. This value must not be nil. |
| options | A combination of the NSKeyValueObservingOptions values that specifies what is included in observation notifications. For possible values, see NSKeyValueObservingOptions. |
| context | Arbitrary data that is passed to anObserver in observeValueForKeyPath:ofObject:change:context:. |
	
observer中需要实现`- observeValueForKeyPath:ofObject:change:context:`才能获得KVO回调。

###44.如何手动触发一个value的KVO  

A:改变值之前调用 `willChangeValueForKey:`，改变值之后调用 `didChangeValueForKey:`。

###45.若一个类有实例变量 `NSString *_foo` ，调用`setValue:forKey:`时，可以以`foo`还是 `_foo` 作为`key`？  

A:都可以。

###46.KVC的keyPath中的集合运算符如何使用？  

A: 

* 必须是集合对象
* KVC支持的集合运算符
* 正确的  Operator key path 格式

###47.KVC和KVO的keyPath一定是属性么？  

A:KVO支持实例变量。

###48.如何关闭默认的KVO的默认实现，并进入自定义的KVO实现？  

A: 

* A class that implements manual notification must override the NSObject implementation of `automaticallyNotifiesObserversForKey:`.For properties that perform manual notification, the subclass implementation of `automaticallyNotifiesObserversForKey:` should return NO. 
* To implement manual observer notification, you invoke `willChangeValueForKey:` before changing the value, and `didChangeValueForKey:` after changing the value. 

###49.Apple用什么方式实现对一个对象的KVO？  

A:
> Automatic key-value observing is implemented using a technique called isa-swizzling.

###50.IBOutlet连出来的视图属性为什么可以被设置成weak?  

A:IBOutlet 连出来的视图是视图层级的组成部分，父视图对它已经有一个强引用，所以它可以被设置为 weak。

###51.IB中User Defined Runtime Attributes如何使用？  

A:
> Add initialization of custom runtime attributes to objects that do not have a corresponding Interface Builder inspector.

###52.如何调试BAD_ACCESS错误  

A:

* Enable Zombie Objects
* Enable Address Sanitizer
* 重写object的respondsToSelector方法，显示出现EXEC_BAD_ACCESS前访问的最后一个object

###53.lldb（gdb）常用的调试命令？  

A: 

```
lldb po
lldb p
lldb expression
lldb bt
lldb c
```

扩展阅读：  
o [上级向的十个iOS面试问题](http://onevcat.com/2013/04/ios-interview/)  
o [百度面试](http://studentdeng.github.io/blog/2014/02/11/baidu-interview/)  

Reference:  
o [iOS Interview](http://blog.sunnyxx.com/2015/07/04/ios-interview/)  
o [iOS Interview Questions](https://github.com/ChenYilong/iOSInterviewQuestions)


