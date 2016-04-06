---
layout: post
title: "iOS App 开发问题汇总（六）"
date: 2016-03-28 20:14:53 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS
discription: iOS development problems
---
### 1. The data couldn’t be read because it isn’t in the correct format.
A:项目是2014年开发的，打包导出 Ad-hoc 时报上面的错误，Build Setting > Enable Bitcode > NO 之后再试成功了。

Reference:http://stackoverflow.com/questions/33995769/ipatool-fails-to-build-with-bitcode-xcode-7-1-1

### 2.setNeedsLayout vs. setNeedsUpdateConstraints and layoutIfNeeded vs updateConstraintsIfNeeded
A:

* If you manipulated constraints directly, call setNeedsLayout.
* If you changed some conditions (like offsets or smth) which would change constraints in your overridden updateConstraints method (a recommended way to change constraints, btw), call setNeedsUpdateConstraints, and most of the time, setNeedsLayout after that.
* If you need any of the actions above to have immediate effect—e.g. when your need to learn new frame height after a layout pass—append it with a layoutIfNeeded.

Reference:http://stackoverflow.com/questions/20609206/setneedslayout-vs-setneedsupdateconstraints-and-layoutifneeded-vs-updateconstra

### 3.父类如何关联子类通过nib初始化的属性？
A: Select xib > Show Utilities > Show The Connection In Inspector > + > Connect to View

### 4. Failed to load test bundle from file

```
Failed to load test bundle from file:///Users/dongmeiliang/Library/Developer/Xcode/DerivedData/JSQMessages-axyqhmblkssajpgcyzizcquhltft/Build/Products/Debug-iphonesimulator/JSQMessages.app/PlugIns/JSQMessagesTests.xctest/../JSQMessagesTests.xctest/: Error Domain=NSCocoaErrorDomain Code=4 "The bundle “$(PRODUCT_NAME)” couldn’t be loaded because its executable couldn’t be located." UserInfo={NSLocalizedFailureReason=The bundle’s executable couldn’t be located., NSLocalizedRecoverySuggestion=Try reinstalling the bundle., NSBundlePath=/Users/dongmeiliang/Library/Developer/Xcode/DerivedData/JSQMessages-axyqhmblkssajpgcyzizcquhltft/Build/Products/Debug-iphonesimulator/JSQMessages.app/PlugIns/JSQMessagesTests.xctest, NSLocalizedDescription=The bundle “$(PRODUCT_NAME)” couldn’t be loaded because its executable couldn’t be located.}
```
A:Tests target > info.plist > delete the row `Bundle name $(PRODUCT_NAME)`;

### 5. pathForResource:ofType:inDirectory:方法有时找不到图片？
A:原因是这个方法必须要指定完整的名字，比如图片的名称为fail_indicator@2x.png，放在JSQMessagesAssets.bundle的images目录下，那可以用如下代码来获取这张图片：

```
+ (NSBundle *)jsq_messagesBundle
{
    return [NSBundle bundleForClass:[JSQMessagesViewController class]];
}

+ (NSBundle *)jsq_messagesAssetBundle
{
    NSString *bundleResourcePath = [NSBundle jsq_messagesBundle].resourcePath;
    NSString *assetPath = [bundleResourcePath stringByAppendingPathComponent:@"JSQMessagesAssets.bundle"];
    return [NSBundle bundleWithPath:assetPath];
}

+ (UIImage *)jsq_imageFromMessagesAssetBundleWithName:(NSString *)name
{
    NSBundle *bundle = [NSBundle jsq_messagesAssetBundle];
    NSString *path = [bundle pathForResource:name ofType:@"png" inDirectory:@"Images"];
    return [UIImage imageWithContentsOfFile:path];
}

[UIImage jsq_imageFromMessagesAssetBundleWithName:@"fail_indicator@2x"];
```



