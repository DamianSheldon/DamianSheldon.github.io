---
layout: post
title: "(翻译)How to Play, Record, and Edit Videos in iOS"
date: 2015-01-22 16:48:25 +0800
comments: true
categories: [Archive, iOS Development]
keywords: Play, Record, Edit, Video, iOS
description: Learn the basics of working with videos on iOS with AV Foundation in this tutorial. You'll play, record, and even do some light video editing!
---

录制视频(并用代码播放它们)是你可以用手机来做的最酷的事情之一，但不是所有的应用都利用了它们。它需要 AVFoundation 框架，AVFoundation 从 Lion(10.7)开始是 OS X 的一部分，Apple 在 2010 年将它加入到 iOS 4.

从那之后 AVFoundation 也有了相当多的成长，现在拥有 100 多个类。本教程覆盖媒体播放和一些轻量级编辑来让你开始使用 AVFoundation.

####AVAsset

它是表示基于时间的视听媒体的抽象类例如视频和音频。每个assert包含用于展示或处理的记录集合，可以是任意一种通用媒体类型，包括但不仅限于音频，视频，文本，closed captions 和 subtitles。

一个AVAsset对象定义了定义记录的属性集合，它构成asset。一个记录用一个AVAssetTrack实例表示。

在典型的简单情况下，一个记录代表一个音频组件，另一个代表视频组件；在复杂的合成器中，可能会有多个重叠的音频和视频记录。你将会把合成的视频和音频文件表示成AVAsset对象。

####AVComposition
An AVCompositionobject combines media data from multiple file-based sources in a custom temporal arrangement in order to present or process it together. All file-based audiovisual assets are eligible to be combined, regardless of container type.

一个AVComposition对象用自定义的时间顺序排列混合来自多个基于文件源的媒体使得可以一起显示或处理。所有基于文件的视听asset都可以混合，而不用管容器类型。


At its top level, an AVComposition is a collection of tracks, each presenting media of a specific type such as audio or video, according to a timeline. Each track is represented by an instance of AVCompositionTrack.

在它的上一级，AVComposition是记录的集合，每个依据时间线展示像音频或视频的媒体类型。每个记录由一个AVCompositionTrack实例代表。

AVMutableComposition and AVMutableCompositionTrack

A higher-level interface for constructing compositions is also presented by AVMutableComposition and AVMutableCompositionTrack. These objects offer insertion, removal, and scaling operations without direct manipulation of the trackSegment arrays of composition tracks.

AVMutableComposition and AVMutableCompositionTrack make use of higher-level constructs such as AVAsset and AVAssetTrack. This means the client can make use of the same references to candidate sources that it would have created in order to inspect or preview them prior to inclusion in a composition.

In short, you have an AVMutableComposition and you can add multiple AVMutableCompositionTrack instances to it. Each AVMutableCompositionTrack will have a separate media asset.

And the Rest

In order to apply a CGAffineTransform to a track, you will make use of AVVideoCompositionInstruction and AVVideoComposition. An AVVideoCompositionInstruction object represents an operation to be performed by a compositor. The object contains multiple AVMutableVideoCompositionLayerInstruction objects.

You use an AVVideoCompositionLayerInstruction object to modify the transform and opacity ramps to apply to a given track in an AV composition. 

AVMutableVideoCompositionLayerInstruction is a mutable subclass of AVVideoCompositionLayerInstruction.

An AVVideoComposition object maintains an array of instructions to perform its composition, and an AVMutableVideoComposition object represents a mutable video composition.

Conclusion

To sum it all up:

You have a main AVMutableComposition object that contains multiple AVMutableCompositionTrack instances. Each track represents an asset.

You have AVMutableVideoComposition objects that contain multiple AVMutableVideoCompositionInstructions.

Each AVMutableVideoCompositionInstruction contains multiple AVMutableVideoCompositionLayerInstruction instances.

Each layer instruction is used to apply a certain transform to a given track.

###[原文](http://www.raywenderlich.com/13418/how-to-play-record-edit-videos-in-ios)


