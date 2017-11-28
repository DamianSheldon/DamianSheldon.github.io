---
layout: post
title: "iOS 的绘图系统"
date: 2014-05-19 16:47:22 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS, UIKit, Core Graphics, Core Animation, OpenCL ES
description: iOS的绘图系统
---

在 iOS 开发过程中，有时我们可能需要自己用代码绘制应用的部分内容，这时候就需要和绘图系统打交道了。 iOS 的绘图系统很强大，我们可以用它来做很多事情；另一方面 Apple 做了很多艰苦卓绝的工作，使得大多数情况下我们可以很轻松完成绘制任务。

大多数人应该都有过绘画的经历，这和 iOS 中的绘画很类似，我相信最开始研究计算机图形学的先驱肯定汲取了很多绘画的精髓，所以我们联系绘画来理解 iOS 中的绘画会很帮助。

当我们在现实生活中绘画时，我们会思考些什么？有些事情是肯定会要考虑的，比如，要画什么内容？画到哪里？用什么东西画?怎么画？在 iOS 中绘画，我们同样要考虑这些问题，只不过这时要和 Apple 定义的术语联系起来，因为我们在她定义的世界里，所以就要按她的规则来做事。  

要画什么内容是由应用的需求决定的，所以不在我们讨论的范围，我们重点关注画到哪里?用什么东西画？怎么画这些问题。iOS 上图形软件技术堆栈大致是这样的：UIKit graphics > Core Graphics(Quartz 2D), Core Animation > OpenGL ES。

Open GL ES 是移动版本的 OpenGL, 它能实现高性能的 2D 和 3D 图形绘制，它不是 Apple 的成果，却是 iOS 图形技术的基石。Apple 在它之上抽象封装了 Core Graphics, Core Animation 等库。大多数应用开发者主要和 2D 图形打交道，所以我们主要使用 Core Graphics，Core Animation。Core Animation 主要为应用提供动画支持，UIKit graphics 的功能相对简单，所以这里我们聚焦 Core Graphics，也称为 Quartz。

上面也可以用来回答用什么画这个问题，即我们可以用 UIKit graphics, Core Animation, OpenGL ES 画，如果我们主要是绘制 2D 图形，那么使用 Core Graphics 是比较正确的选择。

##画到哪里
画到哪里在 iOS 中对应 Graphic Context，我们不妨翻译为图形上下文。在 iOS 应用中有如下可用的图形上下文：bitmap graphics context，PDF graphics context，window graphics context 和 layer context。

##怎么画
在 iOS 中绘画通常是先拿到一个图形上下文，然后配置它的状态，之后给它添加绘图元素，还可以给绘图元素添加效果，最后绘制或填充绘图元素。使用 layer context 绘制时稍微有点区别，它是先绘制到 layer context,之后可以使用这个整体去绘制，有点类似印章盖印。

###图形上下文

####window graphics context

在 iOS 应用中得到 window graphics context 的方法是继承 UIView, 并实现 `drawRect:` 方法，在该方法里调用 UIGraphicsGetCurrentContext 函数。

####bitmap graphics context

UIGraphicsBeginImageContextWithOptions 和 CGBitmapContextCreate 都可以创建 bitmap graphics context, 我们应该优先使用 UIGraphicsBeginImageContextWithOptions， 它的抽象程度更高。

####PDF graphics context

CGPDFContextCreateWithURL 和 CGPDFContextCreate 可以用来创建 PDF graphics context.

####layer context

layer context 要用已有的 graphics context 调用 CGLayerCreateWithContext 来创建。

###图形上下文状态

| 参数 | 备注 |
| :---: | :----: |
| Current transformation matrix (CTM) | Transforms |
| Clipping area | Paths |
| Line: width, join, cap, dash, miter limit | Paths |
| Accuracy of curve estimation (flatness) | Paths |
| Anti-aliasing setting | Graphics Contexts |
| Color: fill and stroke settings | Color and Color Spaces |
| Alpha value (transparency) | Color and Color Spaces |
| Rendering intent | Color and Color Spaces |
| Color space: fill and stroke settings | Color and Color Spaces |
| Text: font, font size, character spacing, text drawing mode | Text |
| Blend mode | Paths and Bitmap Images and Image Masks |


###绘图元素

点，线，圆弧，曲线，路径，椭圆和矩形是提供的绘图元素，可以用它们组合出我们的目标图形。

###效果

效果有阴影，渐变，透明图层。

####阴影
Follow these steps to paint with shadows:
	
1. Save the graphics state.
2. Call the function CGContextSetShadow, passing the appropriate values.
3. Perform all the drawing to which you want to apply shadows.
4. Restore the graphics state.
	
Follow these steps to paint with colored shadows:

1. Save the graphics state.
2. Create a CGColorSpace object to ensure that Quartz interprets the shadow color values correctly.
3. Create a CGColor object that specifies the shadow color you want to use.
4. Call the function CGContextSetShadowWithColor, passing the appropriate values.
5. Perform all the drawing to which you want to apply shadows.
6. Restore the graphics state.
	
####渐变

渐变有线性渐变和角度渐变。CGGradient 和 CGShading 都可以用来实现添加渐变，CGGradient 粗放点，但我们可以少写点代码；CGShading 更细腻，可以根据具体情况选择使用。

####透明图层

Painting to a transparency layer requires three steps:

1. Call the function CGContextBeginTransparencyLayer.
2. Draw the items you want to composite in the transparency layer.
3. Call the function CGContextEndTransparencyLayer.

###绘制、填充

Core Graphics 提供了很多绘制和填充函数，它们的名字通常包含 stroke 或 fill，调用它们可以完成绘制和填充。 

<!-- more -->

##Drawing to a View Graphics Context in iOS

1. Implement drawRect:method;  
2. Mark the view you want update by invocate setNeedDisplay;  
3. Obtain Graphic context by Call UIGraphicsGetCurrentContext method;  
4. Use UIKit provides functions, UIBezierPath or Core Graphics to meet your need.  

``` objective-c
// 1) UIKit method
UIBezierPath *path = [UIBezierPath bezierPath];
[path moveToPoint:CGPointMake(16.72, 7.22)];
[path addLineToPoint:CGPointMake(3.29, 20.83)];
[path addLineToPoint:CGPointMake(0.4, 18.05)];
[path addLineToPoint:CGPointMake(18.8, -0.47)];
[path addLineToPoint:CGPointMake(37.21, 18.05)];
[path addLineToPoint:CGPointMake(34.31, 20.83)];
[path addLineToPoint:CGPointMake(20.88, 7.22)];
[path addLineToPoint:CGPointMake(20.88, 42.18)];
[path addLineToPoint:CGPointMake(16.72, 42.18)];
[path addLineToPoint:CGPointMake(16.72, 7.22)];
[path closePath];
path.lineWidth = 1;
[[UIColor redColor] setStroke];
[path stroke];

// 2) Core Graphics method
CGContextBeginPath(ctx);
CGContextMoveToPoint(ctx, 16.72, 7.22);
CGContextAddLineToPoint(ctx, 3.29, 20.83);
CGContextAddLineToPoint(ctx, 0.4, 18.05);
CGContextAddLineToPoint(ctx, 18.8, -0.47);
CGContextAddLineToPoint(ctx, 37.21, 18.05);
CGContextAddLineToPoint(ctx, 34.31, 20.83);
CGContextAddLineToPoint(ctx, 20.88, 7.22);
CGContextAddLineToPoint(ctx, 20.88, 42.18);
CGContextAddLineToPoint(ctx, 16.72, 42.18);
CGContextAddLineToPoint(ctx, 16.72, 7.22);
CGContextClosePath(ctx);
CGContextSetLineWidth(ctx, 1);
CGContextSetStrokeColorWithColor(ctx, [UIColor redColor].CGColor);
CGContextStrokePath(ctx);
```

##Drawing to a Layer(CGLayer)
A layer context (CGLayerRef) is an offscreen drawing destination associated with another graphics context. It is designed for optimal performance when drawing the layer to the graphics context that created it. A layer context can be a much better choice for offscreen drawing than a bitmap graphics context.

1. “Create a CGLayer Object Initialized with an Existing Graphics Context”  
2. “Get a Graphics Context for the Layer”  
3. “Draw to the CGLayer Graphics Context”  
4. “Draw the Layer to the Destination Graphics Context”  

##Drawing to a BitMap
A bitmap graphics context accepts a pointer to a memory buffer that contains storage space for the bitmap. When you paint into the bitmap graphics context, the buffer is updated. After you release the graphics context, you have a fully updated bitmap in the pixel format you specify.

1. Creating a Bitmap Graphics Context;  
UIGraphicsBeginImageContextWithOptions() or CGBitmapContextCeate()  
2. Draw code.  

``` objective-c
// 1) Mix call UIKit and Core Graphics

UIGraphicsBeginImageContextWithOptions(CGSizeMake(45, 45), YES, 2);
CGContextRef ctx = UIGraphicsGetCurrentContext();
CGContextBeginPath(ctx);
CGContextMoveToPoint(ctx, 16.72, 7.22);
CGContextAddLineToPoint(ctx, 3.29, 20.83);
...
CGContextStrokePath(ctx);
UIGraphicsEndImageContext();

// 2) Core Graphics
CGContextRef ctx = CGBitmapContextCreate(NULL, 90, 90, 8, 90 * 4, space, bitmapInfo);
CGContextScaleCTM(ctx, 0.5, 0.5);
UIGraphicsPushContext(ctx);
UIBezierPath *path = [UIBezierPath bezierPath];
[path moveToPoint:CGPointMake(16.72, 7.22)];
[path addLineToPoint:CGPointMake(3.29, 20.83)];
...
[path stroke];
UIGraphicsPopContext(ctx);
CGContextRelease(ctx);
```
##Concurrency Drawing
``` objective-c
UIImageView *view; // assume we have this
NSOperationQueue *renderQueue; // assume we have this
CGSize size = view.bounds.size;
[renderQueue addOperationWithBlock:^(){
        UIImage *image = [renderer renderInImageOfSize:size];
        [[NSOperationQueue mainQueue] addOperationWithBlock:^(){
            view.image = image;
        }];
}];

- (UIImage *)renderInImageOfSize:(CGSize)size;
{
    UIGraphicsBeginImageContextWithOptions(size, NO, 0);

    // do drawing here

    UIImage *result = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    return result;
}
```

##Reference
o Quartz 2D Programming Guide  
o [绘制像素到屏幕上](http://objccn.io/issue-3-1/)

