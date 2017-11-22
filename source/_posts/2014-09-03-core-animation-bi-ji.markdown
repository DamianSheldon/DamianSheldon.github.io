---
layout: post
title: "Core Animation 笔记"
date: 2014-09-03 09:59:58 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Core Animation, iOS Development
description: Summary of Core Animation
---
##Core Animation介绍
> Core Animation is a graphics rendering and animation infrastructure available on both iOS and OS X that you use to animate the views and other visual elements of your app. -- Apple

Core Animation是iOS和OS X上的图形渲染和动画的基础，你可以用它为视图以及应用的其他可见元素加上动画。  

##什么时候使用Core Animation? 
> In places where you want to perform more sophisticated animations, or animations not supported by the UIView class, you can use Core Animation and the view’s underlying layer to create the animation. Because view and layer objects are intricately linked together, changes to a view’s layer affect the view itself.

在你想要执行更加复杂的动画，或者UIView类不支持的动画时，你可以使用Core Animation和视图底下的layer去创建动画。因为view和layer错综复杂地联系在一起，改变视图的layer会影响视图本身。

UIView内置可动画的属性如下表：  

| Property | Changes you can make                                                                                       |
| :--------: | :---------------------------------------------------------------------------------------------------------  |
| frame    | Modify this property to change the view’s size and position relative to its superview’s coordinate system. 
| bounds   | Modify this property to change the view’s size. 
| center   | Modify this property to change the view’s position relative to its superview’s coordinate system. 
| transform| Modify this property to scale, rotate, or translate the view relative to its center point. 
| alpha    | Modify this property to gradually change the transparency of the view. 
| backgroundColor | Modify this property to change the view’s background color. 
| contentStretch | Modify this property to change the way the view’s contents are stretched to fill the available space. 

<!-- more -->

##如何使用Core Animation?

Core Animation 支持三种动画：一、基于属性的动画；二、关键帧动画；三、Transition 动画；

###基于属性的动画(Property-based animation)

图层的大多数属性是支持动画的，完整列表可以查看 Core Animation Programming Guide 附录 B 中的 Animateable Properties.

我们可以隐式或显示触发属性动画，直接改变支持动画的图层属性就可以触发隐式动画。

```
theLayer.opacity = 0.0;
```

隐式动画使用的是默认的时机控制和动画参数，我们也可以修改这种默认行为。Core Animation 使用 action 对象来为图层实现隐式动画行为，所以我们可以通过返回不同的 action 对象来修改默认行为。

Action 对象遵守 CAAction 协议并定义了一些在图层上执行的行为。所有的 CAAnimation 对象都实现了该协议，当图层属性改变时通常就是运行这些对象。因此我们通过返回 CAAnimation 对象或者自定义 Action 对象来修改隐式动画的默认行为。

那么我们应该何时何处返回 action 对象呢？ Core Animation 是按如下顺序来查找 action 对象的：

1. 如果图层有代理并且代理实现了 `actionForLayer:forKey:` 方法，图层调用该方法。代理必须做下列事情之一：
		
	* 为指定键返回 action 对象。
	* 如果不处理这个动作则返回 nil，这种情况搜索还会继续。
	* 返回 NSNull 对象，这种情况搜索立即停止。
		
2. 图层在 actions 字典里查找指定的键。
3. 图层在 style 字典里查找包含键的 actions 字典。（换名话说，style 字典包含 actions 键，它的值也是字典。图层在第二个字典里查找指定的键。）
4. 图层调用 defaultActionForKey: 类方法。
5. 图层执行 Core Animation 定义的隐式 action（如果有）。

你在哪里安装 action 对象取决于你想怎样修改图层：
	
* 对于那些只在特定环境下使用的 actions ，或者图层已经使用代理对象，提供一个代理并实现 `actionForLayer:forKey:` 方法。
* 对于那些通常不使用代码的图层对象，添加 action 到它的 actions 字典中。
* 对于那些关联你图层自定义属性的 action,包含 action 到图层的 style 字典。
* 对于那些图层的基础行为 action ,继承图层并覆盖 `defaultActionForKey:` 方法。

可以通过创建和配置 CABasicAnimation 对象，然后通过 `addAnimation:forKey:` 执行显式动画。

```
CABasicAnimation* fadeAnim = [CABasicAnimation animationWithKeyPath:@"opacity"];
fadeAnim.fromValue = [NSNumber numberWithFloat:1.0];
fadeAnim.toValue = [NSNumber numberWithFloat:0.0];
fadeAnim.duration = 1.0;
[theLayer addAnimation:fadeAnim forKey:@"opacity"];

// Change the actual data value in the layer to the final value.
theLayer.opacity = 0.0;
```

###关键帧动画(Keyframe animation)

属性动画是从开始值到结束值改变属性，关键帧动画对象可以线性或非线性指定一系列目标值。换句话说，就是我们可以更加细腻地控制动画，可以指定什么时间到达什么数据值。

```
// create a CGPath that implements two arcs (a bounce)
CGMutablePathRef thePath = CGPathCreateMutable();
CGPathMoveToPoint(thePath,NULL,74.0,74.0);
CGPathAddCurveToPoint(thePath,NULL,74.0,500.0,
                                   320.0,500.0,
                                   320.0,74.0);

CGPathAddCurveToPoint(thePath,NULL,320.0,500.0,
                                   566.0,500.0,
                                   566.0,74.0);

CAKeyframeAnimation * theAnimation;

// Create the animation object, specifying the position property as the key path.
theAnimation=[CAKeyframeAnimation animationWithKeyPath:@"position"];
theAnimation.path=thePath;
theAnimation.duration=5.0;

// Add the animation to the layer.
[theLayer addAnimation:theAnimation forKey:@"position"];
```

那么怎么指定什么时间到达什么数据值呢？可以通过指定关键帧值和它的时间安排来完成。

####指定关键帧值

关键帧值是关键帧动画最重要的部分。这些值定义了动画执行路径的行为。主要方法通过一个包含对象的数组，但是对于包含 CGPoint 数据类型（例如图层的 anchorPoint 和 position）的值，可以用 CGPathRef 数据类型代替。  

当指定数组时，它里面的内容取决于属性必须的数据类型。你可以直接添加某些对象到数组中；但是某些对象在添加前必须转换为 id ,所有标量类型和结构体必须用对象包裹。例如：

* 对于接受 CGRect 的属性（例如 bounds 和 frame 属性），用 NSValue 包裹每个矩形。
* 对于图层的 transform 属性，用 NSValue 包裹每个 CATransform3D 矩阵。关键帧动画会轮流应用每个变换矩阵来动画这个属性。
* 对于 borderColor 属性，添加到数组前将每个 CGColorRef 数据类型转换成 id 类型。
* 对于接受 CGFloat 的属性，用 NSNumber 包裹每个值。
* 当动画图层的 contents 属性时，指定一个 CGImageRef 数组。

对于接受 CGPoint 数据类型的属性，你可以创建一个 points 数组或者你可以使用 CGPathRef 对象来指定跟随的路径。当你指定一个点的数组，关键帧动画对象在两点间绘制一条直线并跟随路径。当指定一个 CGPathRef 对象，动画从路径的起点开始并跟随它的轮廓，包括任意曲线表面。你可以使用开环或闭环路径。

####指定关键帧动画的时间安排

关键帧动画的时间安排和节奏比简单动画要更加复杂，你可以使用多个属性来控制它：

* calculationMode 属性定义了用来计算动画时间安排的算法。它的值影响如何使用其他时间安排相关属性。

	* Linear 和 cubic 动画的 calculationMode 被设置为 kCAAnimationLinear 或 kCAAnimationCubic，使用提供的时间安排信息来产生动画。这些模式给你对动画时间安排最大的控制。
	
	* Paced 动画的 calculationMode 被设置为 kCAAnimationPaced 和 kCAAnimationCubicPaced。它不依赖 keyTimes 或 timingFunctions 属性提供外部的时间安排值。相反，是用一个常量速度隐式计算时间安排值提供给动画。
	
	* Discrete 动画的 calculationMode 被设置为 kCAAnimationDiscrete ,动画的属性从一个关键帧值跳跃到另一个值而不会有任何插值。该计算模式使用 keyTimes 属性中的值但是忽略 timingFunctions 属性。
	
* keyTimes 属性指定应用到每个关键帧值的时间标记。只有计算模式为 kCAAnimationLinear, kCAAnimationDiscrete, 或 kCAAnimationCubic 时才使用该属性。 Paced 动画不使用。

* timingFunctions 属性指定每段关键帧动画使用的时间安排曲线。（该属性替换继承的 timingFunctions 属性）如果你想自己处理动画时间安排，使用 kCAAnimationLinear 或 kCAAnimationCubic 模式， keyTimes 和 timingFunctions 属性。keyTimes 定义每个关键帧值的时间点。timing functions 控制所有的中间时间值，它允许你对每段应用 ease-in 或 ease-out 曲线。如果你不指定任何时间安排函数，时间安排就是线性的。

###Transition animation

Transition 动画对象为图层创建动画的可视过渡。它最常见的用途是展示一个图层的同时隐藏另一个图层。和只改变图层一个属性的基于属性的动画不同，过渡动画操纵图层缓存的图片来创建可视效果，这对于改变动画属性很难或者不可能。标准类型的过渡可以让执行 reveal，push，move，或者 crossfade 动画。在 OS X 上，你也可以使用 Core Image filters 来创建过渡，它使用其他的效果，例如 wipes, page curls, ripples 或者你设计的自定义效果。

你可以创建 CATransition 对象并添加到过渡相关的图层上来执行过渡动画。使用 transition 对象来指定过渡的类型以及动画的开始和结束点。你并不需要使用完整的过渡动画。 动画时 transition 对象让你指定使用的开始和结束进度值。这些值让你可以在中间开始或结束动画。

```
CATransition* transition = [CATransition animation];
transition.startProgress = 0;
transition.endProgress = 1.0;
transition.type = kCATransitionPush;
transition.subtype = kCATransitionFromRight;
transition.duration = 1.0;

// Add the transition animation to both layers
[myView1.layer addAnimation:transition forKey:@"transition"];
[myView2.layer addAnimation:transition forKey:@"transition"];

// Finally, change the visibility of the layers.
myView1.hidden = YES;
myView2.hidden = NO;
```

###动画组

上面我们看到都是单个的动画，有的时候我们可能相同时执行多个动画，Core Animation 提供了两种方法：一是 CAAnimationGroup；二是嵌套 transition；

用 CAAnimationGroup 同时执行两个动画：

```
// Animation 1
CAKeyframeAnimation* widthAnim = [CAKeyframeAnimation animationWithKeyPath:@"borderWidth"];
NSArray* widthValues = [NSArray arrayWithObjects:@1.0, @10.0, @5.0, @30.0, @0.5, @15.0, @2.0, @50.0, @0.0, nil];
widthAnim.values = widthValues;
widthAnim.calculationMode = kCAAnimationPaced;

// Animation 2
CAKeyframeAnimation* colorAnim = [CAKeyframeAnimation animationWithKeyPath:@"borderColor"];
NSArray* colorValues = [NSArray arrayWithObjects:(id)[UIColor greenColor].CGColor,
            (id)[UIColor redColor].CGColor, (id)[UIColor blueColor].CGColor,  nil];
colorAnim.values = colorValues;
colorAnim.calculationMode = kCAAnimationPaced;

// Animation group
CAAnimationGroup* group = [CAAnimationGroup animation];
group.animations = [NSArray arrayWithObjects:colorAnim, widthAnim, nil];
group.duration = 5.0;

[myLayer addAnimation:group forKey:@"BorderChanges"];
```

嵌套显式过渡动画

```
[CATransaction begin]; // Outer transaction

// Change the animation duration to two seconds
[CATransaction setValue:[NSNumber numberWithFloat:2.0f]
                forKey:kCATransactionAnimationDuration];
// Move the layer to a new position
theLayer.position = CGPointMake(0.0,0.0);

[CATransaction begin]; // Inner transaction

// Change the animation duration to five seconds
[CATransaction setValue:[NSNumber numberWithFloat:5.0f]
                 forKey:kCATransactionAnimationDuration];

// Change the zPosition and opacity
theLayer.zPosition=200.0;
theLayer.opacity=0.0;

[CATransaction commit]; // Inner transaction

[CATransaction commit]; // Outer transaction
```

###图层自定义属性的动画

上面提到的动画都是针对图层的可动画属性的，Core Animation 也支持给自定义属性添加动画。

CAAnimation 和 CALayer 类扩展了 key-value coding 惯例来支持自定义属性。你可以用这种行为添加数据到图层然后通过你定义的键获取它。你甚至可以关联你的自定义属性的 actions，这样你改变属性时会执行相应的动画。

如果你想用 `actionForLayer:forKey:` 提供自定义属性相关的 action，不要实现相应的 setter 方法，不然会导致 `actionForLayer:forKey:` 不被调用。

如果想通过 style 字典提供自定义属性相关的 action，可以这么做：

```
- (id)init
{
    if ((self = [super init])) {
        
        NSMutableDictionary *style = [NSMutableDictionary dictionaryWithDictionary:self.style];

        NSDictionary *action = @{ClockFaceTimeKey: self.rotationAction};

        [style setObject:action forKey:@"actions"];
        
        self.style = style;
    }
    return self;
}
```

### 停止显式动画

停止显式动画有两种方法：

* 调用图层的 `removeAnimationForKey:` 方法移除你的动画对象，可以从图层移除单个动画对象。该方法使用的是传入 `addAnimation:forKey:` 方法的键来标识动画。你指定的键不能为 nil.  

* 调用图层的 `removeAllAnimations` 方法可以从图层移除所有的动画对象。该方法立即移除所有进行的动画并用当前的状态信息重绘图层。  

### Core Animation 主要类和协议的关系

Core Animation提供了不少类供我们在应用中使用，下图反映了这些类的关系：   
<div style="text-align:center" markdown="1">

<img name="animations_info_2x" src="/images/animations_info_2x.png" width="552" height="446">  

</div>

###Reference

* Core Animation Programing Guide   
* [动画解释](http://objccn.io/issue-12-1/)    
* [Layer 中自定义属性的动画](http://objccn.io/issue-12-2/)


