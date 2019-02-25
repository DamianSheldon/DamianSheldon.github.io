---
layout: post
title: "Auto Layout"
date: 2014-10-19 17:43:12 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Auto Layout, 自动布局, iOS, Adaptive, Mutiple Screen, iPhone 6, iPhone 6 Plus
description: Introduce auto layout, build adaptive user interface with auto layout 
---

###前言 
Auto Layout 早在 iOS 6时就引入了，但由于之前 iPhone 的尺寸不多，而且宽度是一样的; 另一方面 Auto Layout增加了学习成本，大部分开发者仍然使用传统坐标布局做屏幕适配。但是随着 iPhone 6, 6 Plus 大屏 iPhone的发布，继续使用坐标布局做适配显得力不从心了，而且从 Apple 的动作来看，Auto Layout是未来的必然趋势，因此，我们很有必要掌握它。  

###Auto Layout 是什么   
> Auto Layout is a system that lets you lay out your app’s user interface by creating a mathematical description of the relationships between the elements. —Apple  

Auto Layout是一个通过创建元素之间关系的数学描述来布局你应用的用户界面的系统。

> You define these relationships in terms of constraints either on individual elements, or between sets of elements.  

你可以在单个元素，或一系列元素间以约束的形式来定义这些关系。

####Constraint(约束)  
约束是 Auto Layout 的基石，它表达着界面元素布局的规则。我们可以把约束想像成人类语言表述的数学形式。例如，设计师可能会说“这个按钮的左边缘应该与容器视图的左边缘有20个点的偏移。”，它可以转化为button.left = (container.left + 20)，进而抽象出更一般的表达，y = m*x + b，这就是约束。这里的 y 和 x 是 View 的 attributes，m 和 b 是浮点值。在 Auto Layout 中，她们的取值和含义可以简述如下：  

  * attributes 有left, right, top, bottom, leading, trailing, width, height, centerX, centerY 和 baseline；
  * m 表示倍率；   
  * b 实际上是 Constant value, 是物理大小的偏移；  
  * =，是 Relation, Auto Layout 支持 <=, = , >= 三种关系；  
  * Priority level, 约束还支持优先级，优先级高的先满足。  

####Intrinsic Content Size
有的视图在给定内容后就确定了自己的大小，我们把这种大小称为：Intrinsic Content Size。 例如，按钮的大小是基于它的标题加上四周的留白。Auto Layout 是在每个尺寸上用一对约束来表示 Intrinsic Content Size。这一对约束称为 content hugging 和 compression resistance，简称 CHCR, 我们可以通过调整关联元素的 CHCR 的大小来实现她们布局容器的大小调整时谁缩小谁放大。   

Auto Layout 是 Apple 对图形界面元素布局给出的解决方案，她是一种自然语言的布局表达，非常自然清切，个人觉得她比 Web 的布局要强势，可以指哪放哪。这是使用过程中的一个感受，但并不代表 Auto Layout 就比 Web 的布局方法要好，只能说各有千秋吧。  

在布局过程中，界面上的元素可以看成一棵视图树，为了确定布局，视图的位置和大小都要确定，位置都是通过约束来建立，一般就是和父视图建立约束，根视图则是和窗体。视图的大小则可以通过 Intrinsic Content Size 或 约束建立。基于这些输入，Auto Layout 就可以计算出最终的布局信息。  
<!--more-->
###如何使用Auto Layout   
Auto Layout 的使用方法有两种：一是通过 Interface Builder, 二是 Code。在 IB 中是通过拖拽的方式来建立约束。Code 则是通过调用相应的 API 来实现，具体可以参看如下示例。

###Demo
[Auto Layout Demo](https://github.com/DamianSheldon/AutoLayout)  

###Masonry
用代码建立约束时，使用 Apple 提供的 API 代码量还不少，于是就有了 Masonry。她想解决的核心问题就是想让代码布局的表达更简洁高效。目前来看，她的做法还挺受开发者的欢迎，我们可以按需选择使用，毕竟学习她，以后在项目中维护她也是要花时间的啊，只是看这个时间比用 Apple 的 API 完成布局的时间哪个更划算。由于她被广泛使用，所以我们很有可能会要和她打交道，所以这里对它做个简要的使用总结，方便日后查阅。  

Masonry 通过 MASConstraintMaker 来收集 MASConstraint，全部收集完了再建立约束。MASConstraint 是 Masonry 对约束表达的封装，也是 Masonry 实现链式调用的关键，它有两个字类: MASViewConstaint 和 MASCompositeConstraint。 Masonry 还用 MASViewAttribute 封装一侧约束的 View 和 Attribute，来辅助 MASConstraint 的工作。  

建立约束的情况大致有如下四大类：

```
[view mas_makeConstraints:^(MASConstraintMaker *make) {
    // 1
    make.left.equalTo(superview.mas_left);
    make.top.equalTo(superview.mas_top).with.offset(padding.top); //with is an optional semantic filler
    
    //2
    make.edges.equalTo(superview).with.insets(padding);
    
    //3
    make.left.right.equalTo(superview);
}];
```

情况一是直接通过 MASConstraintMaker 来建立单一约束，此时的约束是 MASViewConstaint 对象，由 MASViewAttribute 来封装约束相关的视图和属性，创建的 MASViewConstaint 对象保存在 MASConstraintMaker 的 constraints 中，MASConstraintMaker 安装约束时就是将方向传递给 constraints 的每个 MASViewConstaint，最终实现建立约束。   

情况二是直接通过 MASConstraintMaker 来建立复合约束，此时的约束是 MASCompositeConstraint 对象，它同样保存在 constraints 中，只是她本身是由多个 MASViewConstaint 组成的，这里是组成(Composite)设计模式的典型应用。

情况三是通过链式调用来建立约束，由 MASConstraintMaker 开始，先是生成一个表达 left 的 MASViewConstaint 保存在 constraints 中，right 调用会在此基础上生成一个 MASCompositeConstraint ，她包括 left 和 right 约束，并会替换 constraints 中表达的 left 的 MASViewConstaint，如果再继续链式调用的话，后续的约束都会添加到 MASCompositeConstraint 的 childrenConstraints 中。  

在表达关系时，我们可以使用如下四类值：  

* MASViewAttribute
* UIView/NSView
* NSNumber (Support autoboxing) 
* NSArray

```
make.centerX.lessThanOrEqualTo(view2.mas_left);// MASViewAttribute
    
make.left.greaterThanOrEqualTo(label);// UIView/NSView

make.width.greaterThanOrEqualTo(@200);// NSNumber

// NSArray
make.height.equalTo(@[view1.mas_height, view2.mas_height]);
make.height.equalTo(@[view1, view2]);
make.left.equalTo(@[view1, @100, view3.right]);
```

###Tips  
Xcode 自带布局效果预览的功能，可以按下面的步骤最大化预览编辑窗口查看在各个屏幕上的布局效果： 

  * 在工程导航面板中单击 storyboard/XIB 文件，使它在 Xcode 的主窗口中打开；  
  * 双击上述文件使它在新的窗口中打开；   
  * 将新的窗口移动到新的桌面上，最大化它；   
  * 单击下新窗口，确保它是输入焦点，然后按 Option+Command+Enter 在窗口中打开 assistant editor；    
  * 在 assistant editor 头部选中 Automatic 展开下拉菜单，在下拉菜单中选中 Preview, 可以在 Preview 中一次些查看多个布局效果图，省的来回启动模拟器。  

###Reference  

* Auto Layout Guide  
* WWDC2012 session 202 – Introduction to Auto Layout for iOS and OS X  
* WWDC2012 session 228 – Best Practices for Mastering Auto Layout  
* WWDC2012 session 232 – Auto Layout by Example  
* [Beginning Auto Layout Tutorial in iOS 7: Part 1](http://www.raywenderlich.com/50317/beginning-auto-layout-tutorial-in-ios-7-part-1)   
* [Beginning Auto Layout Tutorial in iOS 7: Part 2](http://www.raywenderlich.com/50319/beginning-auto-layout-tutorial-in-ios-7-part-2)    
* [先进的自动布局工具箱](http://objccn.io/issue-3-5/)    
* [AutoLayout 相关概念介绍和动画demo](http://studentdeng.github.io/blog/2014/06/13/auto-layout/)    
* [WWDC 2012 Session笔记——202, 228, 232 AutoLayout（自动布局）入门](http://www.onevcat.com/2012/09/autoayout/)  


