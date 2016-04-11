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

### 6.自动布局的视图加上遮罩不生效而且视图上UIImageView的图片也不显示了,具体方法如下：

```
- (void)jsq_maskView:(UIView *)view withImage:(UIImage *)image
{
    NSParameterAssert(view != nil);
    NSParameterAssert(image != nil);
    
    UIImageView *imageViewMask = [[UIImageView alloc] initWithImage:image];
    imageViewMask.frame = CGRectInset(view.frame, 2.0f, 2.0f);
    
    view.layer.mask = imageViewMask.layer;
}
```

A:先设置视图的frame。

### 7. 设置视图的frame后，视图最终的坐标和设置的坐标不一致
A:问题的原因是将translatesAutoresizingMaskIntoConstraints设置NO，然后再去设置视图的坐标，这样修改的坐标不会生效，解决办法就是使用坐标是不要将translatesAutoresizingMaskIntoConstraints设置为NO.

### 8.设置视图遮罩时，遮罩以宿主视图的origin为原点，所以当宿主视图的origin不是(0, 0)时，设置遮罩的frame时要考虑将宿主视图的origin设置为(0, 0)之后再进行伸缩，最终得到遮罩的frame。
A:

```
- (void)jsq_maskView:(UIView *)view withImage:(UIImage *)image
{
    NSParameterAssert(view != nil);
    NSParameterAssert(image != nil);
    
    CGRect baseFrameForMask = view.frame;
    baseFrameForMask.origin = CGPointZero;
    
    UIImageView *imageViewMask = [[UIImageView alloc] initWithImage:image];
    imageViewMask.frame = CGRectInset(baseFrameForMask, 2.0f, 2.0f);

    view.layer.mask = imageViewMask.layer;
}
```

### 9.先调用方法二，然后调用方法一，最终的得到图片并没有水平翻转。

```
// 1.
- (UIImage *)jsq_imageMaskedWithColor:(UIColor *)maskColor
{
    NSParameterAssert(maskColor != nil);
    
    CGRect imageRect = CGRectMake(0.0f, 0.0f, self.size.width, self.size.height);
    UIImage *newImage = nil;
    
    UIGraphicsBeginImageContextWithOptions(imageRect.size, NO, self.scale);
    {
        CGContextRef context = UIGraphicsGetCurrentContext();
        
        CGContextScaleCTM(context, 1.0f, -1.0f);
        CGContextTranslateCTM(context, 0.0f, -(imageRect.size.height));
        
        CGContextClipToMask(context, imageRect, self.CGImage);
        CGContextSetFillColorWithColor(context, maskColor.CGColor);
        CGContextFillRect(context, imageRect);
        
        newImage = UIGraphicsGetImageFromCurrentImageContext();
    }
    UIGraphicsEndImageContext();
    
    return newImage;
}

// 2.
+ (UIImage *)imageWithCGImage:(CGImageRef)imageRef
                        scale:(CGFloat)scale
                  orientation:(UIImageOrientation)orientation
```

Code Listing:

```
UIImage *originImage = /* Get origin image*/

UIImage *upMirroredImage = [UIImage imageWithCGImage:originImage.CGImage scale:originImage.scale orientation:UIImageOrientationUpMirrored];

UIImage *upMirroredImageWithMaskedColor = [upMirroredImage jsq_imageMaskedWithColor:[UIColor grayColor]];
```

A:调试中发现`+ (UIImage *)imageWithCGImage:(CGImageRef)imageRef scale:(CGFloat)scale orientation:(UIImageOrientation)orientation` 方法应该没有直接操作图片，而是更改了图片的metadata, 然后`- (UIImage *)jsq_imageMaskedWithColor:(UIColor *)maskColor`是直接操作图片时没有考虑图片的metadata,于是最终得到是原图加上指定颜色遮罩的图片。



