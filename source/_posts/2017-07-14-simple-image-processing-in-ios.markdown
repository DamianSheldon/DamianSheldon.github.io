---
layout: post
title: "iOS 中简单的图片处理"
date: 2017-07-14 09:49:40 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Exif, UIImage, ImageIO
description: Simple image processing in iOS
---
在 iOS 应用开发中，我们可能会要对图片进行旋转、缩放和裁剪，在介绍具体方法前，我们有必要先对图片做个大致的了解，这样有助于我们选择合适的方法。

### 图片格式

图片主要有两种格式：一种叫做位图；另一种称之为矢量图。所谓位图，就是把图片看成是由许多像素点组成；而矢量图则是用绘图指令来描述图片。举个例子，圆可以用圆点，半径，线条的粗细和颜色来描述它。从这个例子也可以看出用矢量图来描述风景，人物这样复杂的事物会比较复杂，所以它们通常会用位图来描述。

位图和矢量图也分为很多格式，具体可能查看[Image file formats](https://en.wikipedia.org/wiki/Image_file_formats)

下面我们讨论的是位图，在 iOS 中我们经常打交道的位图格式是 JPG 和 PNG，用来处理位图数据的类有：UIImage (UIKit)，CGImage (Core Graphics) 和 CIImage (Core Image)。Image I/O 本来是属于 Core Graphics，为了更加方便使用，Apple 将它分离出来成为单独的库。

###旋转
既然位图是用一个一个的像素点来模拟图片，当我们想要旋转图片时，首先想到的方法自然是改变这些像素点的位置，这当然可以达到目标。要调整这么多像素点的位置自然要耗费不少时间，所有在数码相机刚出来那会，人们不是去改变像素点的位置，而是用一段数据来描述图片的方向等信息，这段数据称为 Exif. 所以对于 JPG 这种拥有 Exif 信息的位图，我们旋转图片的最佳做法自然是改变 Exif 里的方向信息。而 PNG 是没有 Exif 信息的，所以只能改变像素点的位置。

UIImage 自带了几个可以旋转的方法：

```
+ (UIImage *)imageWithCGImage:(CGImageRef)cgImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation;

+ (UIImage *)imageWithCIImage:(CIImage *)ciImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation;

- (instancetype)initWithCGImage:(CGImageRef)cgImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation;

- (instancetype)initWithCIImage:(CIImage *)ciImage scale:(CGFloat)scale orientation:(UIImageOrientation)orientation;
```
<!--more-->

我们没有这个方法的实现源码，但是我们可以输出图片的 Exif 信息来验证上面的说法。iOS 中我们可以使用 Image I/O 这个库来读取和修改图片的 Exif 信息。Image I/O 的文档不是很详细，使用时最好结合头文件的说明，而且要注意区分容器和单个图片，实验表明像方向这种信息它并不是放在 Exif 中，而是图片属性中，它的值和 UIImageOrientation 也不是一一对应的，它们的关系如下：

```
typedef NS_ENUM(NSInteger, DMLImagePropertyOrientation) {
    DMLImagePropertyOrientationUp               = 1,
    DMLImagePropertyOrientationDown             = 3,
    DMLImagePropertyOrientationLeft             = 8,
    DMLImagePropertyOrientationRight            = 6,
    DMLImagePropertyOrientationUpMirrored       = 2,
    DMLImagePropertyOrientationDownMirrored     = 4,
    DMLImagePropertyOrientationLeftMirrored     = 5,
    DMLImagePropertyOrientationRightMirrored    = 7
};

+ (DMLImagePropertyOrientation)dml_imagePropertyOrientationFromUIImageOrientation:(UIImageOrientation)imageOrientation
{
    DMLImagePropertyOrientation imagePropertyOrientation = DMLImagePropertyOrientationUp;
    
    switch (imageOrientation) {
        case UIImageOrientationUp:
            imagePropertyOrientation = DMLImagePropertyOrientationUp;
            break;
            
        case UIImageOrientationDown:
            imagePropertyOrientation = DMLImagePropertyOrientationDown;
            break;
            
        case UIImageOrientationLeft:
            imagePropertyOrientation = DMLImagePropertyOrientationLeft;
            break;
            
        case UIImageOrientationRight:
            imagePropertyOrientation = DMLImagePropertyOrientationRight;
            break;
            
        case UIImageOrientationUpMirrored:
            imagePropertyOrientation = DMLImagePropertyOrientationUpMirrored;
            break;
            
        case UIImageOrientationDownMirrored:
            imagePropertyOrientation = DMLImagePropertyOrientationDownMirrored;
            break;
            
        case UIImageOrientationLeftMirrored:
            imagePropertyOrientation = DMLImagePropertyOrientationLeftMirrored;
            break;
            
        case UIImageOrientationRightMirrored:
            imagePropertyOrientation = DMLImagePropertyOrientationRightMirrored;
            break;
    }
    
    return imagePropertyOrientation;
}
```

从打印输出的内容看出 JPG 图片的方向确实改变了，然后我也写了个方法去改变图片的方向属性，得到了同样的效果，所以当我们是旋转上面提到的方向直接使用 UIImage 自带的几个旋转的方法应该是最佳选择，而要旋转任意角度，还是要通过调整像素点位置来完成。

```
// 输出图片的属性信息
- (UIImageOrientation)dml_imageOrientationFromExif
{
    UIImageOrientation imageOrientation = UIImageOrientationRightMirrored + 1;
    
    NSData *dataOfImage = UIImageJPEGRepresentation(self, (CGFloat)0.7);
    
    CGImageSourceRef imageSource = CGImageSourceCreateWithData((__bridge CFDataRef)dataOfImage, NULL);
    
    CFDictionaryRef imageProperties = CGImageSourceCopyPropertiesAtIndex(imageSource, 0, NULL);
    
    NSLog(@"dml_imageOrientationFromExif image Properties:%@\n", (__bridge NSDictionary *)imageProperties);
    
    CFRelease(imageSource);
    
    CFNumberRef numberOfImageOrientation = CFDictionaryGetValue(imageProperties, kCGImagePropertyOrientation);
    
    CFRelease(imageProperties);
    
    DMLImagePropertyOrientation imagePropertyOrientation = [(__bridge NSNumber *)numberOfImageOrientation integerValue];
    
    imageOrientation = [[self class] dml_uiimageOrientationFromImagePropertyOrientation:imagePropertyOrientation];
    
    return imageOrientation;
}

// 修改图片的方向属性
- (UIImage *)dml_setExifOritenation:(UIImageOrientation)imageOrientation error:(NSError * __autoreleasing *)error
{
    NSData *dataOfImage = UIImageJPEGRepresentation(self, (CGFloat)0.7);
    
    CGImageSourceRef imageSource = CGImageSourceCreateWithData((__bridge CFDataRef)dataOfImage, NULL);
    
    /* get the file type */
    CFStringRef UTI = CGImageSourceGetType(imageSource);
    if ( NULL == UTI ) {
        /* Handle Error Retrieving File Type Accordingly */
        if (error) {
            *error = [NSError errorWithDomain:(__bridge NSString *)kCFErrorDomainCGImageMetadata code:kCGImageMetadataErrorUnknown userInfo:@{NSLocalizedDescriptionKey: @"Handle Error Retrieving File Type Accordingly"}];
        }
        return nil;
    }
    
//    CFMutableDataRef finalImageData = (__bridge_retained CFMutableDataRef)dataOfImage.mutableCopy;
    CFMutableDataRef finalImageData = (__bridge_retained CFMutableDataRef)[NSMutableData new];

    /* create an image destination for saving the file */
    CGImageDestinationRef destination = CGImageDestinationCreateWithData(finalImageData, UTI, 1, NULL);
    if ( nil == destination ) {
        /* Handle Error Creating CGImageDestinationRef Accordingly */
        if (error) {
            *error = [NSError errorWithDomain:(__bridge NSString *)kCFErrorDomainCGImageMetadata code:kCGImageMetadataErrorUnknown userInfo:@{NSLocalizedDescriptionKey: @"Handle Error Creating CGImageDestinationRef Accordingly"}];
        }
        return nil;
    }
    
    /* setting properties */
//    CFDictionaryRef sourceProperties = CGImageSourceCopyProperties(imageSource, NULL);
    CFDictionaryRef sourceProperties = CGImageSourceCopyPropertiesAtIndex(imageSource, 0, NULL);

    NSLog(@"dml_setExifOritenation original properties:%@\n", (__bridge NSDictionary *)sourceProperties);
    
    CFMutableDictionaryRef mutableSourceProperties = CFDictionaryCreateMutableCopy(kCFAllocatorDefault, CFDictionaryGetCount(sourceProperties) + 1, sourceProperties);
    
    DMLImagePropertyOrientation imagePropertyOrientation = [[self class] dml_imagePropertyOrientationFromUIImageOrientation:imageOrientation];
    
    CFNumberRef numberForOritentation = CFNumberCreate(kCFAllocatorDefault, kCFNumberNSIntegerType, &imagePropertyOrientation);

    CFDictionarySetValue(mutableSourceProperties, kCGImagePropertyOrientation, numberForOritentation);
    
    NSLog(@"dml_setExifOritenation edited properties:%@\n", (__bridge NSDictionary *)mutableSourceProperties);
    
    CGImageDestinationAddImageFromSource(destination, imageSource, 0, mutableSourceProperties);
    CGImageDestinationFinalize(destination);
    
    UIImage *resultImage = [UIImage imageWithData:(__bridge NSData *)finalImageData];
    
    CFRelease(numberForOritentation);
    
    CFRelease(mutableSourceProperties);
    
    CFRelease(sourceProperties);
    
    // Print destination properties
    
    NSData *dataOfDestinationImage = UIImageJPEGRepresentation(resultImage, (CGFloat)0.7);
    
    CGImageSourceRef destinationImageSource = CGImageSourceCreateWithData((__bridge CFDataRef)dataOfDestinationImage, NULL);
    
    CFDictionaryRef properties = CGImageSourceCopyPropertiesAtIndex(destinationImageSource, 0, NULL);

    NSLog(@"destination properties:%@\n", (__bridge NSDictionary *)properties);
    
    CFRelease(properties);
    CFRelease(destinationImageSource);
    
    return resultImage;
}

```

PS.理论上来讲，我们可以使用 `void CGImageDestinationSetProperties(CGImageDestinationRef idst, CFDictionaryRef properties);` 搭配 `bool CGImageDestinationCopyImageSource(CGImageDestinationRef idst, CGImageSourceRef isrc, CFDictionaryRef options, CFErrorRef  _Nullable *err);` 来改变图片的属性的，实际实验中并没有达到预期效果，原因不明。

###缩放

上面提到 UIImage 的旋转方法也可以指定 Scale 因子，它是我们常说的几倍图中这个几倍因子，如果图片本来是一倍图，然后我们欺骗这个方法说是0.5倍图，那么我们会得到一张放大2倍的图，以些类推，所以我们可以考虑用这个方法来满足我们的一些简单需求，不能满足时，我们可以用 Core Graphics 将图片画到目标大小的位图上下文中来得到我们想要的图片。

###裁剪

UIImage 没有裁剪相关的方法，我们可以使用 Core Graphics 中的 `CGImageRef CGImageCreateWithImageInRect(CGImageRef image, CGRect rect);`方法来实现裁剪。使用这个方法我们需要注意 Rect 要考虑图片的 scale，图片完整的 `rect = {0, 0, image.size.width * image.scale, image.size.height * image.scale}`, 所以我们指定裁剪的 rect 时也要带上 scale.

###缩略图

生成缩略图可以使用 Image I/O 中的 `CGImageRef CGImageSourceCreateThumbnailAtIndex(CGImageSourceRef isrc, size_t index, CFDictionaryRef options);`方法。

###Reference

* Image I/O Programming Guide
* [图片格式](https://objccn.io/issue-21-2/)
* [Image Resizing Techniques](http://nshipster.com/image-resizing/)


