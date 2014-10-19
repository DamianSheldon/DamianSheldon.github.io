---
layout: post
title: "Auto Layout"
date: 2014-10-19 17:43:12 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Auto Layout, 自动布局, iOS, Adaptive, Mutiple Screen, iPhone 6, iPhone 6 Plus
discription: Introduce auto layout, build adaptive user interface with auto layout 
---

###前言 
Auto Layout 早在 iOS 6时就引入了，但由于之前 iPhone 的尺寸不多，而且宽度是一样的; 另一方面 Auto Layout增加了学习成本，大部分开发者仍然使用传统坐标布局做屏幕适配。但是随着 iPhone 6, 6 Plus 大屏 iPhone的发布，继续使用坐标布局做适配显得力不从心了，而且从 Apple 的动作来看，Auto Layout是未来的必然趋势，因此，我们很有必要掌握它。  

###Auto Layout 是什么   
  Auto Layout is a system that lets you lay out your app’s user interface by creating a mathematical description of the relationships between the elements. —Apple  
  Auto Layout是一个通过创建元素之间关系的数学描述来布局你应用的用户界面的系统。

  You define these relationships in terms of constraints either on individual elements, or between sets of elements.  
  你可以在单个元素，或一系列元素间以约束的形式来定义这些关系。

####Constraint -- 约束  
  约束是 Auto Layout 的基石，它表达着界面元素布局的规则。我们可以把约束想像成人类语言表述的数学形式。例如，设计师可能会说“这个按钮的左边缘应该与容器视图的左边缘有20个点的偏移。”，它可以转化为button.left = (container.left + 20)，进而抽象出更一般的表达，y = m*x + b，这就是约束。这里的 y 和 x 是View的attributes，m 和 b 是浮点值。  
  * attributes 有left, right, top, bottom, leading, trailing, width, height, centerX, centerY 和 baseline；   
  * b 实际上是 Constant value, 是物理大小的偏移；  
  * =，是 Relation, Auto Layout 支持 <=, = , >= 三种关系；  
  * Priority level, 约束还支持优先级，优先级高的先满足。  

####Intrinsic Content Size
  Intrinsic Content Size 是 Auto Layout 中另一个重要概念，身处视图层级末端的视图会为显示特定内容期望得到一个大小，它就叫做Intrinsic Content Size。

###如何使用Auto Layout   
  Auto Layout 的使用方法有两种：一是通过 Interface Builder, 二是 Code。

###Tips  
  在实际的项目中，由于3.5 到5.5 Inch跨度还是挺大，建议在 ViewController 的 View 上面加一个 UIScrollView， 然后再在 ScrollView 上加一个 View， 其他的视图都布局在它上面，这样布局会容易点。

###Demo
[Auto Layout Demo](https://github.com/DamianSheldon/AutoLayout)

###Reference  
Auto Layout Guide  
WWDC2012 session 202 – Introduction to Auto Layout for iOS and OS X  
WWDC2012 session 228 – Best Practices for Mastering Auto Layout  
WWDC2012 session 232 – Auto Layout by Example  
http://www.raywenderlich.com/50317/beginning-auto-layout-tutorial-in-ios-7-part-1  
http://www.raywenderlich.com/50319/beginning-auto-layout-tutorial-in-ios-7-part-2  
http://objccn.io/issue-3-5/  
http://studentdeng.github.io/blog/2014/06/13/auto-layout/  
http://www.onevcat.com/2012/09/autoayout/