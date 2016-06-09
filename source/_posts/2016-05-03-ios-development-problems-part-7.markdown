---
layout: post
title: "iOS App 开发问题汇总（七）"
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

### 5. How to load image from CocoaPods resource bundle ?
A:

```
  s.resource_bundles = {
    'DMLSelector' => ['DMLSelector/Assets/*.png']
  }
  
  NSBundle *bundle = [NSBundle bundleForClass:[self class]];
  self.imageView.image = [UIImage imageNamed:@"DMLSelector.bundle/Checked" inBundle:bundle compatibleWithTraitCollection:nil];
```

Reference:http://stackoverflow.com/questions/23835052/load-image-from-cocoapods-resource-bundle

### 6. How to specify NSLineBreakMode in boundingRectWithSize?
A: Use NSParagraphStyleAttributeName & NSParagraphStyle:

```
NSMutableParagraphStyle *paragraph = [[NSMutableParagraphStyle alloc] init];
paragraph.lineBreakMode = NSLineBreakByWordWrapping; //e.g.

CGSize size = [label.text boundingRectWithSize: constrainedSize options:NSStringDrawingUsesLineFragmentOrigin attributes: @{ NSFontAttributeName: label.font, NSParagraphStyleAttributeName: paragraph } context: nil].size;
```
Reference:http://stackoverflow.com/questions/20631464/how-to-specify-nslinebreakmode-in-boundingrectwithsize

### 7.

```
INFO [2016-06-01 15:42:30.78]: ---------------------------------------------------------------------
INFO [2016-06-01 15:42:30.78]: --- Step: fir p ../BBCApp.ipa -T token ---
INFO [2016-06-01 15:42:30.78]: ---------------------------------------------------------------------
INFO [2016-06-01 15:42:30.78]: $ fir p ../BBCApp.ipa -T c977500277789d01cc67d82750057858
INFO [2016-06-01 15:42:34.31]: ▸ /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/dependency.rb:296:in `to_specs': Could not find 'fir-cli' (>= 0) among 206 total gem(s) (Gem::LoadError)
INFO [2016-06-01 15:42:34.31]: ▸ from /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/dependency.rb:307:in `to_spec'
INFO [2016-06-01 15:42:34.31]: ▸ from /System/Library/Frameworks/Ruby.framework/Versions/2.0/usr/lib/ruby/2.0.0/rubygems/core_ext/kernel_gem.rb:47:in `gem'
INFO [2016-06-01 15:42:34.31]: ▸ from /usr/local/bin/fir:22:in `<main>'
```

A: 问题的原因是找不到fir这个命令，可以直接指定绝对路径解决。

```
sh "/Library/Ruby/Gems/2.0.0/gems/fir-cli-1.5.0/bin/fir p ../BBCApp.ipa -T token"
```

### 8. 怎么获取iPhone上已安装App的ipa文件？
A: 可以利用iTunes下载已购的功能。

You can redownload apps on your iPhone or iPod touch, iPad, Mac or PC, or Apple TV (4th generation). You can also redownload some in-app purchases.

On a Mac or PC
From iTunes
1. Open iTunes.
2. Click Sign In, and then enter your Apple ID and password.
3. Click your name and select Purchased from the menu.
4. From the upper-right corner of the screen, click Apps.
5. Click "Not in My Library" to view purchased content that isn't on your computer.
6. Scroll to find the item that you want to download.
7. Click the Download icon  in the upper-right corner of the item that you want to download. Your app downloads to your library.

下载需要点时间，所以点开My Apps tab面不能马上看到刚下载的App,右上角有个下载进度提示，点击可以看详情。

Reference:https://support.apple.com/en-qa/HT201272

### 9. UICollectionView's cellForItemAtIndexPath is not being called
A: 

```
self.automaticallyAdjustsScrollViewInsets = NO;
```

Reference:http://stackoverflow.com/questions/14668781/uicollectionviews-cellforitematindexpath-is-not-being-called?rq=1

### 10. iOS supported audio formats?
A:

```
@"aac",@"adts",@"ac3",@"aif",@"aiff",@"aifc",@"caf",@"mp3",@"mp4",@"m4a",@"snd",@"au",@"sd2",@"wav"
```

Reference:http://stackoverflow.com/questions/1761460/supported-audio-file-formats-in-iphone
https://www.raywenderlich.com/69365/audio-tutorial-ios-file-data-formats-2014-edition

### 11. How do I put the image on the right side of the text in a UIButton?

A:

```
CGFloat spacing          = 3;
    CGFloat insetAmount      = 0.5 * spacing;

    // First set overall size of the button:
    button.contentEdgeInsets = UIEdgeInsetsMake(0, insetAmount, 0, insetAmount);
    [button sizeToFit];

    // Then adjust title and image insets so image is flipped to the right and there is spacing between title and image:
    button.titleEdgeInsets   = UIEdgeInsetsMake(0, -button.imageView.frame.size.width - insetAmount, 0,  button.imageView.frame.size.width  + insetAmount);
    button.imageEdgeInsets   = UIEdgeInsetsMake(0, button.titleLabel.frame.size.width + insetAmount, 0, -button.titleLabel.frame.size.width - insetAmount);
```

Reference:http://stackoverflow.com/questions/7100976/how-do-i-put-the-image-on-the-right-side-of-the-text-in-a-uibutton

### 12. duplicate symbol _iToastDuration

```
enum iToastDuration {
	iToastDurationLong = 10000,
	iToastDurationShort = 1000,
	iToastDurationNormal = 3000
}iToastDuration;
```
A: 

```
typedef enum iToastDuration {
	iToastDurationLong = 10000,
	iToastDurationShort = 1000,
	iToastDurationNormal = 3000
}iToastDuration;
```

