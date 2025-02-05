---
layout: post
title: "(翻译)Property Values"
date: 2014-05-10 14:41:40 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Property Valule, Objective-C, iOS
description: Introduce Objective-C's Property
---
 
  现场教学的一部分乐趣是应付问题和独立思考。在我展示了一些在类中广泛使用的 @property 代码，其中一个学生问到"@property 后面繁琐的字段都是什么？我已经使用了 retain, assign 和 nonatomic，但我真的不知道它们真正是干什么用的。"（如果你仅仅只是想要一份快速参考，文章最后有所有的 @property 属性。）
  
  对于许多 Objective-C 程序员来说，@property 就像一个神奇的护身符。添加一些像这样的代码给你的狗对象一个名字：
  
```
@property (copy, nonatomic) NSString *puppyName;

```

把它放到 Xcode 的代码片断库中，当需要它时粘贴进来然后编辑它。

@property 是一个属性声明，它有两个目的：声明一个或两个方法，当你使用这些方法时简洁地描述它们的运行时语义。
  
##我声明

@property 声明是声明访问器方法的简写。这些方法让你从对象获取值（嘿 puppy，你叫什么名字？）或者改变值（puppy 欢迎回家，现在你的名字是 Rumperlstiltskin）。下面就是类声明中的 property 声明:

```
@interface Puppy: NSObject
@property NSString *puppyName;
@end
```
<!--more-->
编译器将表现得像你实际上声明了两个方法:

```
@interface Puppy : NSObject
- (NSString *) puppyName;
- (void) setPuppyName: (NSString *) newPuppyName;
@end
```

它使用的惯例是：用 property 名字做 getter 方法，在 property 名字上添加 set 作为 setter 方法，除非另有说明。

##Nom Nom Nomenclature

如果你不想让 setter 和 getter 使用默认名字？一些 true/false 属性有一个像 "isHousebroken" 比 "housebroken"读起来更好。你可以添加一些 API 控制到你的 property 描述:

```
@property (getter=isHousebroken, setter=setHousebrokenness:) BOOL housebroken;
```

这告诉编译器，即使属性的名字是 "housebroken"，也使用 `-isHousebroken`方法而不是 `-housebroken`来获取值，用 `-setHousebrokenness:`而不是 `-setHousebroken`改变值。假设你有一个 puppy 对象，你可以得到它的 housebroken 状态：

```
if ([puppy isHousebroken]) ... do stuff
```

和改变它:

```
[puppy setHousebrokenness: YES];
```

你也可以用点语法使用属性名:

```
puppy.housebroken = YES;
```

点语法纯粹是普通对象消息发送的语法糖。编译器看到你正在使用 housebroken 属性。它也知道 `-setHousebrokenness:` setter 被调用了，所以编译器实际发射这个：

```
[puppy setHousebrokenness: YES];
```

##并发

iOS 开发者们总是看到 `nonatomic` 关键字。它是 @property 咒语和舞蹈的一部分--如果你创建一个属性，让它 nonatomic。但是这意味着什么？它是 atomic 的反义(噗)。

但是什么是 `atomic`？它意味着线程安全吗？

这取决于你说的线程安全是什么。在某种意义上，atomic 属性是本地线程安全--atomic 值可以被多个线程改变而不会损坏因为读写属性值是串行的。

说 puppy 得到了一个可以自由漫步的后院，你想存放院子的区域：

```
@property CGRect domain;
```

属性默认是 atomic。这意味着如果多个线程同时操纵值你不会得到垃圾。矩形在内存中看起来像这样：
<div style="text-align:center" markdown="1">
                                                                                           <img name="Puppy Rect 1" src="/images/puppyrect-1.png">
                                                                                        </div>

四个 CGFloat 端对端堆叠。现在你有两个线程想改变它：

```
<b>thread 1:</b> puppy.domain = CGRectMake (1.0, 2.0, 3.0, 4.0);
<b>thread 2:</b> puppy.domain = CGRectMake (10.0, 20.0, 30.0, 40.0);
```

atomic 意味着在竞争条件赋值时你将得到结果：？
<div style="text-align:center" markdown="1">
                                                                                           <img name="Puppy Rect 3" src="/images/puppyrect-3.png">
                                                                                        </div>

或者这个结果：
<div style="text-align:center" markdown="1">
                                                                                           <img name="Puppy Rect 4" src="/images/puppyrect-4.png">
                                                                                        </div>

但不是乱码：
<div style="text-align:center" markdown="1">
                                                                                           <img name="Puppy Rect 2" src="/images/puppyrect-2.png">
                                                                                        </div>

Nonatomic 属性可能导致最后的乱码，如果值在多个线程中改变。

但是从更大的视角来看，atomic 属性没有让你的代码真正线程安全。它所意味的是单个属性的值总是被完全改变，所以你不会得到新旧混合。

puppy 可能有一连贯的域，但是这剩下的 puppy 数据是错误的。你在一个方法里有设置一些 puppy 属性的代码:

```
puppy.name = @"Hoover";
puppy.domain = CGRectMake (1.0, 2.0, 3.0, 4.0);
puppy.housebroken = NO;
```

另一个方法运行在另一个线程同时做类似的事情，但是是以不同的数据：

```
puppy.name = @"Rumpelstiltskin";
puppy.domain = CGRectMake (10.0, 20.0, 30.0, 40.0);
puppy.housebroken = YES;
```

域是 atomic ，所以它将总是有连贯的值。但是取决于线程怎么被调度，你可能得到一个名叫 Hoover，housebroke 是 No， 但是 domain 是 {10.0, 20.0, 30.0, 40.0}。值是内部一致的，但是是错的。总体来看，这个在 puppy 上的操作不是线程安全的。

让对象属性 atomic 怎么样？ 仍然不是意味着访问对象是线程安全的。你可能有一个 kennel club 对象是这么声明的：

```
@interface KennelKlub : NSObject
@property (atomic) Puppy *alphaPuppy;
@end
```

这不意味着 alphaPuppy 是任意线程安全的。仅仅指针有 atomic 语义。声明仅意味如果两个线程尝试复制4字节32位的指针不会有任何内存干扰。

让代码 atomic 比只是复制一些字节要慢一些。你需要一些同步机制。如果你知道你是在对象值不能在多线程中改变（例如用户界面对象，或者你的对象的合约是"不要在多个线程中使用一个对象"）你可以避免整个同步。

通常的推荐是在 Mac 上使用 atomic 属性，因为机器是如此的快以致于这点同步不是问题。在 iOS 上，用 nonatomic 属性来减少工作的总数，释放有限的 CPU 用作其他用途（对电池也很有好处）。

默认是 atomic，所以如果你想你可以忽略 attribute，但是如果你想显示表明“是的，这个 property 是 atomic 的”使用 atomic 关键字是欢迎的。Clang 早期的版本不支持 atomic 关键字，但是在 Xcode 4 的某个时候添加了。

##小心你的后面

大多数属性有一个对应的实例变量，它为属性保存值。Puppy 对象应该包含一个 NSString 指针用于 puppy 的名字，一个 BOOL 用于 housebroken 状态。这些实例变量来自哪里，它如何被使用?

如果你绝对没做其他事，只是在你的类或扩展中有一个 @property 语句，编译器将添加一个下划线后紧跟属性名的实例变量到你的代码。在当前的情况下，我们将自动得到 `_puppyName` 和 `_houseBroken`。为什么前导下划线？它[防止特定类别的 bug](https://www.bignerdranch.com/blog/a-motivation-for-ivar-decorations/)。 编译器还会为 getter 和 setter 生成代码。

你可以提供你自己的 setter 或者 getter 方法(或者两者)。如果任意一个方法源码没有提供，编译器将提供一个实现。例如：

```
@property (nonatomic) NSInteger split;
...
- (NSInteger) split {
    NSLog (@"Someone called split!  Value is %zd", _split);
    return _split;
}
```

你提供 getter。编译器将提供 setter，还添加后备实例变量 `_split`。

但是有几个边界情况。

如果你既提供 setter 也提供了 getter ，编译器假设你处理所有的细节包括存储属性值到哪里。这种情况编译器不会为你产生后备实例变量。幸运的是有[三个地方](https://www.bignerdranch.com/blog/where-does-it-go-george/)你可以声明你自己的实例变量。你可以显示的 @synthesize 它们。确保尊重你代码中 property attribute 声明的合同。

当你实现你自己的访问器方法时atomic 属性添加皱纹。当编译器生成实现 setter 和 getter 代码时它查找属性是否包含 nonatomic attribute：

```
@property (nonatomic) CGRect domain;
```

编译器将只会生成从一个地方移动 sizeof(CGRect) 字节到另一个地方而不尝试同步数据的代码。但是如果你有一个 atomic 属性:

```
@property CGRect domain;
```

setter 和 getter 都必须同意同步机制。如果你有一个 atomic 属性，仅只提供一个 setter 或 getter 方法，编译器将产生一些警告([你会修复警告](https://www.bignerdranch.com/blog/a-bit-on-warnings/)，对不对？)因为它不知道你实际上是怎么做同步的。

```
puppeh.m:38:1: warning: writable atomic property 'domain' cannot pair a synthesized setter
      with a user defined getter [-Watomic-property-with-user-defined-accessor]
- (CGRect) domain {
^
puppeh.m:38:1: note: setter and getter must both be synthesized, or both be user defined, or the
      property must be nonatomic
puppeh.m:14:30: note: property declared here
@property CGRect domain;
```

## 可改变性

属性可以是只读的。你对象可能有些属性实际上是计算得到的值。Puppy 有名字和姓，全名是它们两者的连接:

```
@property (nonatomic, copy) NSString *firstName;
@property (nonatomic, copy) NSString *lastName;
@property (nonatomic, readonly) NSString *puppyName;
...
- (NSString *) puppyName {
    return [NSString stringWithFormat: @"%@ %@", self.firstName, self.lastName];
}
```

通过使用只读属性 attribute，你告诉编译器仅仅生成 `-puppyName`方法。如果你尝试调用 setPuppyName: 编译器会有趣地看着你。这意味着类的用户可以获取值，但是不能直接改变。

对于 puppyName 这里浪费了一个自动合成实例变量空间吗？没有。当你显式为只读属性提供访问器方法，编译器聪明到知道这时没有必要为它创建一个实例变量。它只是之前关于编译器何时创建自动合成后备实例变量规则的一个应用。

你可以在类的公共接口声明属性为只读，在类扩展时重新声明为读写。这让你在你的实现中使用编译器生成的 atomic setter 和 getter 方法，但是不暴露 setter 方法给余下的世界。查看[关于读/写的全部](https://www.bignerdranch.com/blog/readwrite-all-about-it/)了解更多详情。

##内存管理

Cocoa 对于内存管理有很多选择。非对象值以字节方式复制。对象指针可以是 strong，它意味着指向的对象将被保持存活。你可以有 weak 对象指针，它不保持其他对象存活，如果指向的对象销毁了它会被回填为零。你可以复制对象而不再引用原始对象。你也可以有只进行字节复制的对象指针而不进行内存管理。

你应该总是为 NSString attribute 使用复制。这个的动机可以在[关于可改变性](https://www.bignerdranch.com/blog/about-mutability/) 的 "Mutable Stripping" 节找到。

在 ARC 下，没有任何修饰默认的内存管理是 strong。

显示声明用于属性的实例变量必须匹配内存管理类型，否则会得到编译器错误。下面是一个(隐式) strong 实例变量将被用于一个 weak 属性：

```
@interface Puppy : NSObject {
    NSString *_favoriteFood;
}
@property (weak) NSString * favoriteFood;
@end
```

得到漂亮的消息：

```
error: existing instance variable '_favoriteFood' for __weak property 'favoriteFood' must be __weak
puppeh.m:22:35: note: property declared here
@property (weak) NSString *favoriteFood
                           ^
```

这是因为实例变量默认是 strong 的。为什么这是一个硬性的错误而不仅是一个警告？在实现文件中如果你混合和匹配直接实例变量访问(strong 指针)并通过属性方法(weak 指针)提供，为单独一个属性使用混合的内存管理概念容易导致混淆。所以要么不要显式声明实例变量，要么用 _weak 前导。

代理和父指针应该是 weak 引用，这样你可以避免循环保留。

有些时候你想对指针使用 assign 语义，例如引用少数一些不能有 weak 引用的类。你可以使用 assign 或者 语义更明显的 unsafe_unretained attribute。你可以在 [Unsafe at Any Speed]()查看关于 unsafe_unretained 的更多内容。

## 夸张的 ARC 转折

如果你只使用 ARC，你可以跳过这一部分。

非 ARC 内存管理不使用 strong 和 weak，而是使用 retain 和 assign attribute 做类似的事情。当然，你仍然可以使用 copy. retain 意味着当赋值时属性会被保留，替换时会被释放。assign 和你之前看到的一样--指针的字节仅被复制，和 unsafe_unretained 一样。但是还没有 weak 引用。如果一个 assigned 对象销毁了，所有指向该对象的引用都成了野引用。在那个地址新建的对象会导致各种奇怪的 bug。Zombies instruments 模板可以帮助定位这种情况。

##TL;DR - Reference

下面是一个所有不同 property attributes 的快速列表:

```
<b>API Control</b>
    getter=methodname
    setter=methodname:

<b>Read / Write Serialization</b> (not general thread safety)
    nonatomic
    atomic (default)

<b>Mutability</b>
    readonly
    readwrite (default)

<b>Memory Management (ARC)</b>
    copy
    strong (default)
    weak
    unsafe_unretained
    assign

<b>Memory Management (Traditional)</b>
    copy
    retain
    assign  (default)
```

##Reference
o [Property Values](http://www.bignerdranch.com/blog/property-values/)

