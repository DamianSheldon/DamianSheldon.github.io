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
  
 Property的attribute关键字有以下几类：  
 1）API Control；
 	getter = methodname
 	setter = mehtodname:
 
 2）Write Serialization(not general thread safety);
 	nonatomic
 	atomic (default)
 	
 3) Mutability;
 	readonly
 	readwrite (default)
 
 4) Memory Management(ARC);
 	copy
 	strong (default)
 	weak
 	unsafe_unretained
 	assign
 
 5) Memory Management(Traditional)
	copy
	retain
	assign (default)

<!-- more -->

##API Control
@property声明实际上是存取方法的快速声明。
``` objective-c
	@property (nonatomic, readwrite, strong) NSObject *propertyName;
```
编辑器最终会合成存取方法：
``` objective-c
	- (NSObject *)propertyName;
	- (void)setPropertyName:(NSObject *)anObject;
```
默认的getter和setter方法名分别是propertyName,setPropertyName。可以通过API Control提供的关键字指定存取方法名。
``` objective-c
@property(nonatomic,getter=isEnabled) BOOL enabled;                                  // default is YES. if NO, ignores touch events and subclasses may draw differently
@property(nonatomic,getter=isSelected) BOOL selected;                                // default is NO may be used by some subclasses or by application
@property(nonatomic,getter=isHighlighted) BOOL highlighted;                          // default is NO. this gets set/cleared 
```

##Write Serialization(not general thread safety)
atomic是指存在竞争赋值时，我们会得到某次完整的赋值，而nonatomic则可能是几次赋值共同组合，它并不是通常所指的线程安全。访问atomic修饰的property会比atomic修饰的property慢，这也很明显，因为要做一些额外的操作确保赋值操作是串行的。

##Mutability
readonly和readwrite容易理解，就是限定property的访问权限。

##Memory Management(ARC)  
1)assign，普通赋值，不更改引用计数。适用于标量等非对象数据类型，如char, int ,float, double, NSUinteger, NSInteger等。 
 
2)copy,在内存分配一块全新的地址来存放传入的数据内容，即创建一份新的数据副本用来赋值。适用于实现了NSCoping协议的对象，其他类型的对象声明copy无效。  

3)strong(=retain)，强引用，ARC下对象默认内存管理声明关键字，对象引用计数+1。  

4)weak,弱引用，持有对象引用计数不变，持有对象释放时，指向的地址为nil。  

5)unsafe_unretained,和weak类似，区别是持有对象释放时成为野指针,访问它会造成程序crash(iOS5.0 Or higher不要使用它)。

##Memory Management(Traditional)

##Reference
o [Property Values](http://www.bignerdranch.com/blog/property-values/)

