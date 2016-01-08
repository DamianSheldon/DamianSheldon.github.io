---
layout: post
title: "iOS Development Problems Part 5"
date: 2016-01-08 21:32:22 +0800
comments: true
categories: [Archives, iOS Development]
keywords: 
discription: 
---
### 1.On iOS 7 and later, how do I take a snapshot of my view and save the result in a UIImage?
Solution:

```
- (UIImage *)snapshot:(UIView *)view
{
    UIGraphicsBeginImageContextWithOptions(view.bounds.size, YES, 0);
    [view drawViewHierarchyInRect:view.bounds afterScreenUpdates:YES];
    UIImage *image = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return image;
}
```
Reference:https://developer.apple.com/library/ios/qa/qa1817/_index.html


