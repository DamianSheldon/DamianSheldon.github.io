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
<!-- more -->
### 4.Settings bundle not showing up in iPhone settings

Solution:iOS 9.0.x - 9.2 Settings App Bug, Using the App Switch UI (double-press Home button) to kill the Settings App.

Reference:http://stackoverflow.com/questions/2683793/settings-bundle-not-showing-up-in-iphone-settings

### 5.单元测试的 Target 在测试导航栏里是灰色的不能点击测试。

Solution: Edit Scheme ... > Test > + > Select your test target.

### 6.How to change UITableViewCell Image to Circle in UITableView
Solution:

```
// Subclass UITableViewCell override layoutSubviews method

- (void)layoutSubviews
{
    [super layoutSubviews];
    
    self.imageView.layer.cornerRadius = CGRectGetHeight(self.imageView.frame) * 0.5;
    self.imageView.layer.masksToBounds = YES;
}
```

Reference:http://stackoverflow.com/questions/23350728/how-to-change-uitableviewcell-image-to-circle-in-uitableview

###7.How to downgrading Cocoapods?
A:

```
// First you can figure out which version of Cocoapods you are on with the command:
$ pod --version

// You can also see all the version of Cocoapods you have installed with this command:
$ sudo gem list cocoapods

// Next uninstall Cocoapods. If you have multiple version, you will have the choice of uninstalling all or a specific version.
$ sudo gem uninstall cocoapods

// Finally you can install the specific version with this command:
$ sudo gem install cocoapods -v 0.39.0
```

Reference:https://diego.org/2015/09/24/downgrading-cocoapods/

###8.How to get UISearchDisplayController to search only after search button is pressed？
A:Setup a delegate for the search bar and implement the searchBarSearchButtonClicked: method. Do your searching from that method. Just return NO from the shouldReloadTableForSearchString method.

Reference:http://stackoverflow.com/questions/18563529/how-to-get-uisearchdisplaycontroller-to-search-only-after-search-button-is-press


