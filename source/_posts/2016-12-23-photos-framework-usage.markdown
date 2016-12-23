---
layout: post
title: "Photos 框架的基本使用"
date: 2016-12-23 14:23:33 +0800
comments: true
categories: [Archives, iOS Development] 
keywords: Photos, PHPhotoLibrary, PHAssetCollection, PHAsset,  Assets Library 
discription: 
---

从 iOS 9 开始 Apple 把 Asset Library 标记为废弃状态，并建议开发者使用 Photos 框架。

>The Assets Library framework is deprecated as of iOS 9.0. Instead, use the Photos framework instead, which in iOS 8.0 and later provides more features and better performance for working with a user’s photo library. 

不幸的是 Apple 并没有发布相关的使用指导文档，只有一个相关 Demo。使用的时候固然可以回头参考这个 Demo，但这样的效率不是很高，很多概念也容易忘记，所以这里做个简单的总结。

Photos 中有不少类，其中几个犹为关键。PHPhotoLibary 是我们操作 Photo Library 里面资源的入口对象，所有的操作都通过它完成。PHCollectionList 表示相册中的专题列表；PHAssetCollection 表示专题；PHAsset 表示资源，如 images, videos, and Live Photos.

我们基本的需求就是 CRUD, 这些操作是需要用户授权的，记得先获取权限再操作， 下面我们展示相关的代码片段。

### Create

1. 创建一个资源

```
PHPhotoLibrary.shared().performChanges({
            PHAssetChangeRequest.creationRequestForAsset(from: image)
        }, completionHandler: {success, error in
            if !success { print("error creating asset: \(error)") }
        })
```

2. 创建一个资源到指定的专题

```
PHPhotoLibrary.shared().performChanges({
            let creationRequest = PHAssetChangeRequest.creationRequestForAsset(from: image)
            if let assetCollection = self.assetCollection {
            let addAssetRequest = PHAssetCollectionChangeRequest(for: assetCollection)
            addAssetRequest?.addAssets([creationRequest.placeholderForCreatedAsset!] as NSArray)
            }
        }, completionHandler: {success, error in
            if !success { print("error creating asset: \(error)") }
        })
```
<!-- more -->
### Read (Fetch)

获取资源是通过 PHAsset 提供的一系列以 fetchXXX 开头的类方法，选择哪个方法取决于需求，这里示例其中两个我觉得常用的方法。

1. `class func fetchAssets(with options: PHFetchOptions?) -> PHFetchResult<PHAsset>`

我们可以用这个方法获取 Photo Library 里面所有的资源。

```
let allPhotosOptions = PHFetchOptions()
    allPhotosOptions.sortDescriptors = [NSSortDescriptor(key: "creationDate", ascending: true)]
self.fetchResult = PHAsset.fetchAssets(with: allPhotosOptions)
```

2. `class func fetchAssets(in assetCollection: PHAssetCollection, options: PHFetchOptions?) -> PHFetchResult<PHAsset>` 

我们可以用这个方法获取指定专题里面的资源。例如我们想获取 Camera Roll 这个专题里面的资源：

```
let cameraRoll: PHFetchResult<PHAssetCollection> = PHAssetCollection.fetchAssetCollections(with: .smartAlbum, subtype: .smartAlbumUserLibrary, options: nil).firstObject
let fetchResult = PHAsset.fetchAssets(in: cameraRoll, options: nil)
```

### Update (Edit)

编辑的基本的做法是先用资源请求一个 PHContentEditingInput，然后编辑资源，为了方便用户之后继续编辑或撤销可以实例化一个 PHAdjustmentData 对象来持有相关信息。编辑完成之后对于图片和视频需要实例化一个 PHContentEditingOutput 来完成输出，PHContentEditingOutput 的 adjustmentData 属性关联之前的 PHAdjustmentData, 并把编辑完成的内容输出到 PHContentEditingOutput 的 renderedContentURL。最后创建一个 PHAssetChangeRequest 对象，设置它的 contentEditingOutput 为
之前实例化的 PHContentEditingOutput。

这部分的代码会多点，具体可以查看 [Demo](https://github.com/DamianSheldon/PhotosFrameworkUsage).

### Delete

```
PHPhotoLibrary.shared().performChanges({ 
        PHAssetChangeRequest.deleteAssets([self.asset] as NSArray)
        }) { (success, error) in
    DispatchQueue.main.sync {
        self.trashButton.isEnabled = success ? false : true
    }

    if success {
        print("delete asset successfully")
    }
    else {
        print("can't delete asset: \(error)")
    }
}
```

### 完整 Demo

[PhotosFrameworkUsage](https://github.com/DamianSheldon/PhotosFrameworkUsage)  

### Caveat

使用过程中遇到一个坑，这里记一下。

```
guard let inputImage = CIImage(contentsOf: input.fullSizeImageURL!)
            else { fatalError("can't load input image to edit") }

// Apply the filter.
let outputImage = inputImage
    .applyingOrientation(input.fullSizeImageOrientation)
.applyingFilter(filterName, withInputParameters: nil)

// List 1.
let uiImage = UIImage(ciImage: outputImage)

// List 2.
if let cgImage = CIContext(options: nil).createCGImage(outputImage, from: outputImage.extent) {
    let uiImage = UIImage(cgImage:cgImage)
}
else {
    print("instance UIImage from CGImage failed!")    
}

// Ouput
if let data = UIImageJPEGRepresentation(uiImage, 0.7) {
    // NSData - (BOOL)writeToURL:(NSURL *)url atomically:(BOOL)atomically;
    do {
        try data.write(to: output.renderedContentURL)

    } catch let error {
        print("output filtered image to specify URL failed: \(error)")
    }
}
else {
    print("generate JPEG representation data failed")
        return
}
```

这里的问题是直接用 CIImage 实例化  UIImage 会失败，得转成 CGImage 然后实例化 UIImage. 至于它的原因我暂时还不清楚。

Reference:[UIImageJPEGRepresentation returns nil](http://stackoverflow.com/questions/29732886/uiimagejpegrepresentation-returns-nil)
