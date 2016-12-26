---
layout: post
title: "iOS 开发问题汇总(九)"
date: 2016-11-15 14:49:17 +0800
comments: true
categories: [Archives, iOS Development] 
keywords: iOS, Swift
discription: 
---
###1.Curried functions in Swift
A:There’s a difference between self.methodname (which you are using), and Classname.methodname.

The former, when called within a class’s method, will give you a function bound with that class instance. So if you call it, it will be called on that instance.

The latter gives you a curried function that takes as an argument any instance of Classname, and returns a new function that is bound to that instance. At this point, that function is like the first case (only you can bind it to any instance you like).

Here’s an example to try and show that a bit better:

```
class C {
    private let _msg: String
        init(msg: String) { _msg = msg }

    func c_print() { print(_msg) }

    func getPrinter() -> ()->() { return self.c_print }
}

let c = C(msg: "woo-hoo")
let f = c.getPrinter()
// f is of type (())->()
f() // prints "woo-hoo"

let d = C(msg: "way-hey")

let g = C.c_print
// g is of type (C) -> (()) -> (),
// you need to feed it a C:
g(c)() // prints "woo-hoo"
g(d)() // prints "way-hey"

// instead of calling immediately,
// you could store the return of g:
let h = g(c)
// at this point, f and h amount to the same thing:
// h is of type (())->()
h() // prints "woo-hoo"
```
Reference:[Curried functions in SWIFT](http://stackoverflow.com/questions/27644702/curried-functions-in-swift)

###2.NSLog on devices in iOS 10 / Xcode 8 will truncate.
A:A temporary solution, just redefine all NSLOG to printf in a global header file.

```
#define NSLog(FORMAT, ...) printf("%s\n", [[NSString stringWithFormat:FORMAT, ##__VA_ARGS__] UTF8String]);
```

Reference:[NSLog on devices in iOS 10 / Xcode 8 seems to truncate? Why？](http://stackoverflow.com/questions/39584707/nslog-on-devices-in-ios-10-xcode-8-seems-to-truncate-why)  

<!--more-->
###3.UIImagePickerViewController is black doesn't work for selecting photo or taking picture.
A:I instance it via `UIImagePickerViewController(nibName:nil, boundle:nil)` that doesn't work, and change to `UIImagePickerViewController()` it works.

###4.How to create dispatch queue in Swift?
A:

```
// Concurrent dispatch queue
let concurrentQueue = DispatchQueue(label: "queuename", attributes:.concurrent)

// Serial dispatch queue
let serialQueue = DispatchQueue(label: "queuename")
```

Reference:[How to create dispatch queue in Swift 3](http://stackoverflow.com/questions/37805885/how-to-create-dispatch-queue-in-swift-3)

###5.How to create a bitmap graphics context in Swift 3?
A:

```
// Create a bitmap graphics context with the sample buffer data
guard let context = CGContext(data: baseAddress, width: width, height: height, bitsPerComponent: 8,
        bytesPerRow: bytesPerRow, space: colorSpace, bitmapInfo: CGImageAlphaInfo.premultipliedFirst.rawValue | CGBitmapInfo.byteOrder32Little.rawValue) else {
    return nil
}
```

Reference: [CGBitmapContextCreate error with swift](http://stackoverflow.com/questions/24109149/cgbitmapcontextcreate-error-with-swift)

###6.Required plug-in compatibility UUID 37B30044-3B14-46BA-ABAA-F01000C27B63 for plug-in at path '~/Library/Application Support/Developer/Shared/Xcode/Plug-ins/Unity4XC.xcplugin' not present in DVTPlugInCompatibilityUUIDs
A:There isn't official document about developing plugin for Xcode, so we can only attempt to solve it. Thanks to the internet, we can get a solution by other figure out.

```
XCODEUUID=`defaults read /Applications/Xcode.app/Contents/Info DVTPlugInCompatibilityUUID`
for f in ~/Library/Application\ Support/Developer/Shared/Xcode/Plug-ins/*; do defaults write "$f/Contents/Info" DVTPlugInCompatibilityUUIDs -array-add $XCODEUUID; done
```

The reason probably is plugin developed compate with old Xcode, so it of course don't contain the lastest Xcode's UUID, we can manual add it if it really compate with the latest Xcode. 

Reference: [Xcode 5 - Required plug-in not present in DVTPlugInCompatibilityUUIDs?](http://stackoverflow.com/questions/20732327/xcode-5-required-plug-in-not-present-in-dvtplugincompatibilityuuids?rq=1)

###7.Why AVCaptureSession output a wrong orientation?
A:AVCaptureVideoPreviewLayer and AVCaptureOutput are different output destination, so we have to set oritentation for them each.

```
let captureConnection = videoDataOutput.connection(withMediaType: AVMediaTypeVideo)

if captureConnection!.isVideoOrientationSupported {
captureConnection!.videoOrientation = AVCaptureVideoOrientation.portrait
}
else {
print("capture connection\(captureConnection!) doesn't support video orientation")
}
```
Reference:[Why AVCaptureSession output a wrong orientation?](http://stackoverflow.com/questions/3561738/why-avcapturesession-output-a-wrong-orientation?rq=1)  
[Technical Q&A QA1744 Setting the orientation of video with AV Foundation](https://developer.apple.com/library/content/qa/qa1744/_index.html)  
