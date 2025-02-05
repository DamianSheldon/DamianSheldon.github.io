---
layout: post
title: "Objective-C Class And Object"
date: 2014-05-10 07:54:58 +0800
comments: true
categories: [Archives,iOS Development]
keywords: Objective-C, Class, Object, Programming, Category, ISA
description: Introduce Objective-C's class and object
---

Objective-C是一门面向对象的编程语言，既然是面向对象，那我们就有必要对它的对象作进一步的理解，而且它的很多特性与这也大有关系。

###1.Class
``` objective-c
	/// An opaque type that represents an Objective-C class.
	typedef struct objc_class *Class;

	struct objc_class {
    Class isa  OBJC_ISA_AVAILABILITY;

	#if !__OBJC2__
    Class super_class                                        	OBJC2_UNAVAILABLE;
    const char *name                                         	OBJC2_UNAVAILABLE;
    long version                                             	OBJC2_UNAVAILABLE;
    long info                                                	OBJC2_UNAVAILABLE;
    long instance_size                                       	OBJC2_UNAVAILABLE;
    struct objc_ivar_list *ivars                             	OBJC2_UNAVAILABLE;
    struct objc_method_list **methodLists                    	OBJC2_UNAVAILABLE;
    struct objc_cache *cache                                 	OBJC2_UNAVAILABLE;
    struct objc_protocol_list *protocols                     	OBJC2_UNAVAILABLE;
	#endif

	} OBJC2_UNAVAILABLE;
```

<!-- more -->

###2.Object
``` objective-c 
	/// Represents an instance of a class.
	struct objc_object {
    	Class isa  OBJC_ISA_AVAILABILITY;
	};
```

从头文件的定义可以看出,对象的内存布局以一个Class 类型的isa指针开始,而类的内存布局也是以一个Class类型的isa指针开始，所以类也是对象，这样就满足了面向对象编程语言中所有的东西都是对象。

对象是对象，类也是对象。很容易混淆是吧，所以OC重用了一个术语来区分它们:类对象（Class Object）;实例对象（Instance Object）。实例对象的isa指针指向的类叫Class,类对象的isa指针指向的类叫meta-class。meta-class的isa指针指向根类meta-class,根类meta-class的isa指针指向自己。让我们看张图：
{% img /images/Objective-C_Class_And_Object.png Objective-C Class And Object  %}

那么类对象是怎么创建的呢？经过查找，得到的结论是编译器创建的，编译时所谓的类是指类对象（官方文档： The class object is the compiled version of the class.）。


###3.category

因为对象在内存中的排布可以看成一个结构体，该结构体的大小并不能动态变化。所以无法在运行时动态给对象增加成员变量。
相对的，对象的方法定义都保存在类的可变区域中。Objective-C 2.0并未在头文件中将实现暴露出来，但在Objective-C 1.0中，我们可以看到方法的定义列表是一个名为 methodLists的指针的指针。通过修改该指针指向的指针的值，就可以实现动态地为某一个类增加成员方法。这也是Category实现的原理。同时也说明了为什么Category只可为对象增加成员方法，却不能增加成员变量。


###4.方法混写
因为对象的方法可以改变，因此我们就有了方法混写的技术。

###5.isa混写
除了对象的方法可以动态修改，因为isa本身也只是一个指针，所以我们也可以在运行时动态地修改isa指针的值，达到替换对象整个行为的目的。

典型示例：KVO。

###6.参考资料
1.iOS 6 Programming Pushing the Limits, Rob Napier, Mugunth Kumar  
2.[深入浅出Cocoa之类与对象](http://blog.csdn.net/kesalin/article/details/7211228)  
3.[深入浅出Cocoa 之动态创建类](http://blog.csdn.net/kesalin/article/details/7219572)  
4.[Objective-C对象模型及应用](http://blog.devtang.com/blog/2013/10/15/objective-c-object-model/)  
5.[Objective-C对象之类对象和元类对象（一）](http://blog.csdn.net/wzzvictory/article/details/8592492)  
6.[Objcclass](http://studentdeng.github.io/blog/2011/10/05/objcclass/)


