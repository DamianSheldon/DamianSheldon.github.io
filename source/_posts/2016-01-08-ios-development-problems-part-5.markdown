---
layout: post
title: "iOS App 开发问题汇总（五）"
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

### 2.How to present a view controller on iOS7 without the status bar overlapping?

Solution:The easiest workaround I've found is to wrap the view controller you want to present inside a navigation controller, and then present that navigation controller.

```
MyViewController *vc = [MyViewController new];
UINavigationController *nvc = [[UINavigationController alloc] 
    initWithRootViewController:vc];
[self presentViewController:nvc animated:YES completion:nil];
```

### 3.代码创建 UITableView 时如何使用各种系统样式的 UITableViewCell?
Solution:

```
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath
{
    static NSString *sCellID = @"CellID";

    UITableViewCell *cell = [tableView dequeueReusableCellWithIdentifier:sCellID];
    if (!cell) {
        cell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:sCellID];
    }
    // Configure the cell...
    
    return cell;
}
```

