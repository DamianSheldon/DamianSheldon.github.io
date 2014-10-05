---
layout: post
title: "iOS的绘图系统"
date: 2014-05-19 16:47:22 +0800
comments: true
categories: [Archives, iOS Development]
---
##iOS的绘图框架
1)UIKit是Objective-C形式的API，提供基本的2D图形绘制，图片处理，和一些实现用户界面元素动画的方法；  
 
2)Core Graphics是C形式的API，支持向量图形，位图和PDF内容；  
  
3)Core Animation是另一个Objective-C形式的API,它能为用户界面增加流畅的移动和动态的反馈效果；
  
4)OpenGL ES 是移动端版本的OpenGL,它能实现高性能的2D和3D图形绘制。

##Drawing Destinations(iOS-Only)
1)View Graphics Context  
2)Layer  
3)Bitmap  
4)PDF  
5)Printer  

##Drawing to a View Graphics Context in iOS
1)Implement drawRect:method;  
2)Mark the view you want update by invocate setNeedDisplay;  
3)Obtain Graphic context by Call UIGraphicsGetCurrentContext method;  
4)Use UIKit provides functions, UIBezierPath or Core Graphics to meet your need.  

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

1)“Create a CGLayer Object Initialized with an Existing Graphics Context”  
2)“Get a Graphics Context for the Layer”  
3)“Draw to the CGLayer Graphics Context”  
4)“Draw the Layer to the Destination Graphics Context”  

##Drawing to a BitMap
A bitmap graphics context accepts a pointer to a memory buffer that contains storage space for the bitmap. When you paint into the bitmap graphics context, the buffer is updated. After you release the graphics context, you have a fully updated bitmap in the pixel format you specify.

1)Creating a Bitmap Graphics Context;  
UIGraphicsBeginImageContextWithOptions() or CGBitmapContextCeate()  
2)Draw code.  

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
o 绘制像素到屏幕上 http://objccn.io/issue-3-1/