---
layout: post
title: "输出自定义尺寸视频"
date: 2017-04-10 10:12:09 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Video Resolution, AVFoundation, AVAssetWriter, AVAssetExportSession 
description: How to secify a resolution for output video? 
---
最近要做的一个项目中要拍摄视频，于是就开始来研究视频。看了几遍 AVFoundation Programming Guide 之后也写了个 Demo，把基本功能都过了一遍。这其中有意思的一件事情是我发现微信拍摄短视频的尺寸是 540x944, 这尺寸很奇怪，不是任何一个预设值。不清楚微信为什么用这么一个尺寸，但我想搞清楚怎么输出自定义尺寸的视频。

AVFoundation 捕获数据输出时，各组件的关系如下：

<div style="text-align:center" markdown="1">
                                                                                           <img name="Capture Detail" src="/images/captureDetail_2x.png">
                                                                                        </div>

要想输出自定义尺寸的视频，我们可以从输入端和输出端着手。但是从文档来看，并没有提供可以自定捕获尺寸的方法，所以只能从输出端着手。
<!--more-->

首先我用 AVCaptureMovieFileOutput 做输出，然后调用 `setOutputSettings(_ outputSettings: [AnyHashable : Any]!, for connection: AVCaptureConnection!)` 来达到目标。但是很不幸，控制台输出了异常，查看 AVCaptureMovieFileOutput 的头文件，

> On iOS, you may only specify the AVVideoCodecKey in the outputSettings. If you specify any other key, an NSInvalidArgumentException will be thrown. See the availableVideoCodecTypes property.

所以这个方法行不通。

于是我又尝试用 AVAssetWriter 来接收每一帧捕获的数据，然后按配置输出，理论上来讲这是可行的，实际上只有第一帧数据能成功被接收，之后的数据都会接收失败，具体原因不详。  

直接处理每一帧数据失败之后，我又反复翻阅文档，发现编辑章节中提到可以修改 renderSize, 于是又一个想法冒出来，也许可以通过修改 renderSize 来输出自定义尺寸。按照文档编写好相关代码，激动地运行测试。结果得到的是：

```
Optional(Error Domain=AVFoundationErrorDomain Code=-11800 "The operation could not be completed" UserInfo={NSUnderlyingError=0x1700505c0 {Error Domain=NSOSStatusErrorDomain Code=-12108 "(null)"}, NSLocalizedFailureReason=An unknown error occurred (-12108), NSLocalizedDescription=The operation could not be completed})
```

说实话，内心是崩溃的。但对这件事情还是耿耿于怀，又浏览了一下官方示例列表，发现了 AVSimpleEditoriOS,  

> AVSimpleEditor is a simple AVFoundation based movie editing application which exercises the APIs of AVVideoComposition, AVAudioMix and demonstrates how they can be used for simple video editing tasks. It also demonstrates how they interact with playback (AVPlayerItem) and export (AVAssetExportSession). The application performs trim, rotate, crop, add music, add watermark and export. This sample is ARC-enabled.

嗯，看到里面提到可以裁剪，于是就想它是怎么做？可以剪成我想要的大小吗？阅读相关的代码片断，原来它就是用的 renderSize 来实现裁剪的，跟我第三种方法的代码基本一致，差别是它是 Objc 写的，我用 Swift 写的。既然它能正常工作，那我就用这份代码来输出自定义尺寸吧。把代码移进来，运行测试，居然输出了指定的尺寸，难道代码用 Objc 和 Swift 写还有这种差别，整个人是懵的，这个原因暂时是不清楚的。

虽然输出的尺寸对了，但是画面没有铺满尺寸，而且方向错了。这有点太虐了，既然都走到这一步，就想那我再试试能不手动把它纠正吧。纠正的方法是使用 transform, 但是文档对它的介绍不详细，我先参考了 QuartZ 2D Programming Guide 中 Transforms 来变换，发现不对，整个画面全变成了黑色，又在 AVSimpleEditoriOS 的注释中发现了新的线索，

>Note: the point of origin for rotation is the upper left corner of the composition, t3 is to compensate for origin

这么说它用的坐标和 QuartZ 2D 还不一样啊，这么坑爹，好吧，只能先确定好它们是怎么变换的。于是我先输出一段没变换的视频，之后每次测试一个变换，用这个办法确认了它们的变换是这样的，变换的原点是屏幕的左上角，Translation 向右是 X 轴的正方向，向下是 Y 轴的正方向; Rotation 的度数为正是按顺时针方向旋转，为负则是逆时针方向旋转；Scaling 的值大于1为放大，小于1则是缩小。

这样我就做了这么一个变换：

```
t1 = CGAffineTransformScale(asset.preferredTransform, sx, sy);

t1 = CGAffineTransformRotate(t1, degreesToRadians(90));

t1 = CGAffineTransformTranslate(t1, 540, 0);
```

控制台报错了，说这个视频不支持编辑，这是个什么鬼？完全没有道理啊！于是我又把这段代码从下往上一行一行注释，看是谁导致的问题，发现是 `t1 = CGAffineTransformRotate(t1, degreesToRadians(90));` ，这样我又试着调整变换的顺序，改成：

```
t1 = CGAffineTransformTranslate(asset.preferredTransform, 540, 0);

t1 = CGAffineTransformScale(t1, sx, sy);

t1 = CGAffineTransformRotate(t1, degreesToRadians(90));
```

运行测试，苍天啊，居然可以了。经历这么一出，感觉写代码都成了玄学了, 无力吐槽!
