---
layout: post
title: "Swift Learning Notes -- Initialization"
date: 2014-06-24 11:33:28 +0800
comments: true
published: false
categories: [Archives, iOS Development]
keywords: swift, learn, Initialization, iOS
description: Initialization method of swift
---

###Initialization

“Initialization is the process of preparing an instance of a class, structure, or enumeration for use. This process involves setting an initial value for each stored property on that instance and performing any other setup or initialization that is required before the new instance is ready to for use.” -- Apple Inc.

从苹果的介绍我们可以得出初始化的两个要点:  
1，为实例的每一个存储属性设置初始值;    
2，进行其他必要的准备工作.   

这里我们主要总结下为实例的存储属性设置初始值。Class, Structure, enumeration的初始化略有不同，主要是因为Class可以继承，而这背后的原因是Class是Reference Type,而Structure, enumeration是Value Type。

###Value Type Initialization
1)为存储属性设置默认值；
    Swift会为设置了所有存储属性默认值而没有提供初始化方法的structure提供Default Initializer。
    structure在所有存储属性都设置了默认值的情况下自动接受Memberwise Initializer。

2)初始化方法。
    NOTE: Swift会自动将初始化方法的本地参数名生成外部参数名。

<!-- more -->

###Reference Type Initialization
1)为类引入的存储属性设置初始值;  
2)初始化方法。
    Designated initializers must always delegate up.
    Convenience initializers must always delegate across.

    Swift中子类默认是不继承父类的初始化方法,然而当满足一定条件时例外。
    Assuming that you provide default values for any new properties you introduce in a subclass, the following two rules apply:

    Rule 1
    If your subclass doesn’t define any designated initializers, it automatically inherits all of its superclass designated initializers.

    Rule 2
    If your subclass provides an implementation of all of its superclass designated initializers—either by inheriting them as per rule 1, or by providing a custom implementation as part of its definition—then it automatically inherits all of the superclass convenience initializers.

    These rules apply even if your subclass adds further convenience initializers.

    NOTE:子类可以是用Convenience Initializer实现父类的Designated Initializer.
