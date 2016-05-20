---
layout: post
title: "iOS App 开发问题汇总（六）"
date: 2016-05-03 17:00:04 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS
discription: iOS development problems
---

### 1.如何设置透明的导航栏并且去掉其底部灰色的分隔线？
A:可以通过设置导航栏的背景图和阴隐图为透明的图片来实现。

```
- (void)viewDidAppear:(BOOL)animated
{
    [super viewDidAppear:animated];
    
    self.translucent = self.navigationController.navigationBar.isTranslucent;
    self.navigationController.navigationBar.translucent = YES;
    
    self.previousImage = [self.navigationController.navigationBar backgroundImageForBarMetrics:UIBarMetricsDefault];
    
    UIImage *newImage = [UIImage imageNamed:@"TransparentPixel"];
    [self.navigationController.navigationBar setBackgroundImage:newImage forBarMetrics:UIBarMetricsDefault];
    
    self.previousShadowImage = self.navigationController.navigationBar.shadowImage;
    self.navigationController.navigationBar.shadowImage = newImage;
}

- (void)viewWillDisappear:(BOOL)animated
{
    [super viewWillDisappear:animated];
    
    self.navigationController.navigationBar.translucent = self.translucent;
    
    [self.navigationController.navigationBar setBackgroundImage:self.previousImage forBarMetrics:UIBarMetricsDefault];
    self.navigationController.navigationBar.shadowImage = self.previousShadowImage;
}
```

Reference:Apple's sample code Customizing UINavigationBar  
http://stackoverflow.com/questions/18921082/separator-between-navigation-bar-and-view-ios-7

<!-- more -->

### 2. 自定义的View设置cornerRadius不生效

### 3. CocoaPods 升级到1.0，Xcode 一直报 header file not found.
A: 仔细查看了 Xcode 的编译过程，头文件的路径是包含了的，于是怀疑是Xcode的自身可能存在问题。经过实验，可以用下面的方法解决：

1. Clean project
2. Window > Projects > Your Project > Delete Derived Data
3. Build

### 4. No codesigning identities (i.e. certificate and private key pairs) that match the team ID: xxx
A:

* Go to Xcode preferences, view details of the Apple ID, and delete the provisioning file that's complaining, redownload.
* Go to the Keychain Access, and delete the development certificate that's related to the provisioning file you just deleted.

Reference:http://stackoverflow.com/questions/19197497/ios-7-0-no-code-signing-identities-found


