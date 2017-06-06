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

###8.Operation stopped, the video could not be composed.

```
Domain=AVFoundationErrorDomain Code=-11841 "Operation Stopped" UserInfo=0x1912c320 {NSLocalizedDescription=Operation Stopped, NSLocalizedFailureReason=The video could not be composed.}
```

A: We should instance AVMutableComposition, AVMutableCompositionTrack every time when edit.

```
mutableComposition = AVMutableComposition()

videoCompositionTrack = mutableComposition.addMutableTrack(withMediaType: AVMediaTypeVideo, preferredTrackID: kCMPersistentTrackID_Invalid)

audioCompositionTrack = mutableComposition.addMutableTrack(withMediaType: AVMediaTypeAudio, preferredTrackID: kCMPersistentTrackID_Invalid)
```

###9. How to get photos from iPhone simulator?
A: Photo files are located at :

```
~/Library/Developer/CoreSimulator/Devices/<device UDID>/data/Media/DCIM/100APPLE/
```
with Xcode 8.2. You can get <device UDID> from Devices window or using command:`xcrun simctl list devvices`.

Reference:[getting images from iPhone simulator](http://stackoverflow.com/questions/5488915/getting-images-from-iphone-simulator) 

###10. Where is the default include paths of C headers in macOS?
A: `gcc -x c -v -E /dev/null`  

```
$ gcc -x c -v -E /dev/null

...

#include "..." search starts here:
#include <...> search starts here:
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/../lib/clang/8.0.0/include
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/include
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk/usr/include
/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.12.sdk/System/Library/Frameworks (framework directory)

...

```

Reference:[View default include path of C headers in Mac OS X by `gcc -v`?](http://stackoverflow.com/questions/19852199/view-default-include-path-of-c-headers-in-mac-os-x-by-gcc-v)  

###11. error: include of non-modular header inside framework module

```
JKEncrypt.h:11:9: error: include of non-modular header inside framework module 'JKCategories.NSData_JKEncrypt': '/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator10.3.sdk/usr/include/CommonCrypto/CommonCryptor.h' [-Werror,-Wnon-modular-include-in-framework-module]
#import <CommonCrypto/CommonCryptor.h>
```
A:Add follow instructs to your podspec file.  

```
// Your podspec file 
spec.user_target_xcconfig = {'CLANG_ALLOW_NON_MODULAR_INCLUDES_IN_FRAMEWORK_MODULES' => 'YES'}
```

Reference:[Swift compiler error: “non-modular header inside framework module”](http://stackoverflow.com/questions/24103169/swift-compiler-error-non-modular-header-inside-framework-module)  

###12. Is there a “space” character that is NOT trimmed from end of UILabel?  
A:I haven't found such character yet. Follow sulotion is a workaround:

```
NSMutableAttributedString *string = [[NSMutableAttributedString alloc] initWithString:text attributes:@{NSForegroundColorAttributeName : [UIColor blackColor]}];

[string insertAttributedString:[self emptyAtributedWhitespace] atIndex:0];
[string appendAttributedString:[self emptyAtributedWhitespace]];

label.attributedText = string;

...

- (NSAttributedString *)emptyAtributedWhitespace
{
        // You can put any random string there or how many spaces you want
            return [[NSAttributedString alloc] initWithString:@"_" attributes:@{ NSForegroundColorAttributeName : [UIColor clearColor]}];
}
```

Reference:[Is there a “space” character that is NOT trimmed from end of UILabel?](https://stackoverflow.com/questions/25836307/is-there-a-space-character-that-is-not-trimmed-from-end-of-uilabel)  

###13. 如何更换 SVN 仓库地址？
A:

```
// 1. 查看仓库地址

$ svn info 
Path: .
Working Copy Root Path: /Users/dongmeiliang/Documents/SuiShouPai
URL: http://192.168.2.231:2036/svn/mct_v1/trunk/working/docs/UI/prototype/%E9%9A%8F%E6%89%8B%E6%8B%8D/%E9%9A%8F%E6%89%8B%E6%8B%8DAPP
Relative URL: ^/trunk/working/docs/UI/prototype/%E9%9A%8F%E6%89%8B%E6%8B%8D/%E9%9A%8F%E6%89%8B%E6%8B%8DAPP
Repository Root: http://192.168.2.231:2036/svn/mct_v1
Repository UUID: 59ac94ce-ffd7-5644-ad12-280b91628744
Revision: 12039
Node Kind: directory
Schedule: normal
...

// 2. 更换仓库地址

$ svn relocate http://192.168.2.231:2036/svn/mct_v1/trunk/working/docs/UI/prototype/%E9%9A%8F%E6%89%8B%E6%8B%8D/%E9%9A%8F%E6%89%8B%E6%8B%8DAPP http://192.168.9.21:2036/svn/mct_v1/trunk/working/docs/UI/prototype/%E9%9A%8F%E6%89%8B%E6%8B%8D/%E9%9A%8F%E6%89%8B%E6%8B%8DAPP
```

###14. 在 iOS 中如何计算大文件的 MD5 Hash 值？
A:

```
// Standard library
#include <stdint.h>
#include <stdio.h>

// Core Foundation
#include <CoreFoundation/CoreFoundation.h>

// Cryptography
#include <CommonCrypto/CommonDigest.h>

// In bytes
#define FileHashDefaultChunkSizeForReadingData 4096

// Function
CFStringRef FileMD5HashCreateWithPath(CFStringRef filePath, 
        size_t chunkSizeForReadingData) {

    // Declare needed variables
    CFStringRef result = NULL;
    CFReadStreamRef readStream = NULL;

    // Get the file URL
    CFURLRef fileURL = 
        CFURLCreateWithFileSystemPath(kCFAllocatorDefault, 
                (CFStringRef)filePath, 
                kCFURLPOSIXPathStyle, 
                (Boolean)false);
    if (!fileURL) goto done;

    // Create and open the read stream
    readStream = CFReadStreamCreateWithFile(kCFAllocatorDefault, 
            (CFURLRef)fileURL);
    if (!readStream) goto done;
    bool didSucceed = (bool)CFReadStreamOpen(readStream);
    if (!didSucceed) goto done;

    // Initialize the hash object
    CC_MD5_CTX hashObject;
    CC_MD5_Init(&hashObject);

    // Make sure chunkSizeForReadingData is valid
    if (!chunkSizeForReadingData) {
        chunkSizeForReadingData = FileHashDefaultChunkSizeForReadingData;
    }

    // Feed the data to the hash object
    bool hasMoreData = true;
    while (hasMoreData) {
        uint8_t buffer[chunkSizeForReadingData];
        CFIndex readBytesCount = CFReadStreamRead(readStream, 
                (UInt8 *)buffer, 
                (CFIndex)sizeof(buffer));
        if (readBytesCount == -1) break;
        if (readBytesCount == 0) {
            hasMoreData = false;
            continue;
        }
        CC_MD5_Update(&hashObject, 
                (const void *)buffer, 
                (CC_LONG)readBytesCount);
    }

    // Check if the read operation succeeded
    didSucceed = !hasMoreData;

    // Compute the hash digest
    unsigned char digest[CC_MD5_DIGEST_LENGTH];
    CC_MD5_Final(digest, &hashObject);

    // Abort if the read operation failed
    if (!didSucceed) goto done;

    // Compute the string result
    char hash[2 * sizeof(digest) + 1];
    for (size_t i = 0; i < sizeof(digest); ++i) {
        snprintf(hash + (2 * i), 3, "%02x", (int)(digest[i]));
    }
    result = CFStringCreateWithCString(kCFAllocatorDefault, 
            (const char *)hash, 
            kCFStringEncodingUTF8);

done:

    if (readStream) {
        CFReadStreamClose(readStream);
        CFRelease(readStream);
    }
    if (fileURL) {
        CFRelease(fileURL);
    }
    return result;
}
```

Reference:[Compute MD5 or SHA hash of large file efficiently on iOS and Mac OS X](http://www.joel.lopes-da-silva.com/2010/09/07/compute-md5-or-sha-hash-of-large-file-efficiently-on-ios-and-mac-os-x/)  
