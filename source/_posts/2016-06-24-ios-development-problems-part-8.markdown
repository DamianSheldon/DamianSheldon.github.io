---
layout: post
title: "iOS App 开发问题汇总（八）"
date: 2016-06-24 15:54:30 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS 
discription: iOS development problems
---

### 1.Detect hash tags #, mention tags @, in iOS like in Twitter App
A:You can use https://github.com/Krelborn/KILabel this library. It also detect user taps on hashtags Like this:

```
label.hashtagLinkTapHandler = ^(KILabel *label, NSString *string, NSRange range) {
  NSLog(@"Hashtag tapped %@", string);
};
```

Reference:http://stackoverflow.com/questions/24359972/detect-hash-tags-mention-tags-in-ios-like-in-twitter-app

### 2.如何给应用重签名？
A：利用用途广泛的命令行工具 security快速地显示出你的系统中能用来对代码进行签名的认证的方法：

```
$ security find-identity -v -p codesigning                       
  1) 01C8E9712E9632E6D84EC533827B4478938A3B15 "iPhone Developer: Thomas Kollbach (7TPNXN7G6K)"
```

用[sign](https://github.com/fastlane/fastlane/tree/master/sigh)给应用重签名

```
$ sigh resign ./path/app.ipa --signing_identity "iPhone Distribution: Felix Krause" -p "my.mobileprovision"
```

Reference:http://objccn.io/issue-17-2/

<!--more-->

### 3. It report BAD_ACCESS when set delegate for AVAudioPlayer.

A: The reason is I instance AVAuidoPlayer without content like this:

```
AVAudioPlayer *audioPlayer = [AVAudioPlayer new];
audioPlayer.delegate = self;
```

We should instance it with content:

```
_audioPlayer = [[AVAudioPlayer alloc] initWithData:_audioData error:nil];
[_audioPlayer prepareToPlay];
_audioPlayer.delegate = self;
```

### 4.  How can I show my own app icon at the bottom-left corner of iPhone lock screen?

Solution: 1. This is iOS 8 new feature called Handoff more info [here](http://www.macrumors.com/2014/06/03/ios-8-apps-quick-access/).

2. You can't control what app appears in the bottom-left corner of the lock screen. Neither can you add code to your app to make your app appear there whenever you want. These are called Suggested Apps and it's a feature of iOS.

iOS controls what app appears there based on several factors, including your location, the apps on your device, and your history of where & when you use those apps. Together, iOS trys to present you with the app you would most-likely use at a given time, location, and history.

If you don't like this feature, it can be turned off in Settings -> General -> Handoff & Suggested Apps.

Reference:http://apple.stackexchange.com/questions/166983/how-did-a-calendar-icon-make-it-to-my-lock-screen

http://apple.stackexchange.com/questions/166983/how-did-a-calendar-icon-make-it-to-my-lock-screen

### 5.

```
	 Class AJXorEncryptor is implemented in both /Users/dongmeiliang/Library/Developer/CoreSimulator/Devices/71988A60-BE7B-425A-BDE4-AAE4D098A516/data/Containers/Bundle/Application/051ED3F9-A920-41C8-9D96-41EED06DE618/AJFrameDevApp.app/AJFrameDevApp and /Users/dongmeiliang/Library/Developer/Xcode/DerivedData/AJFrameDevApp-fwmhocnqvmbmzkftzzdryfyigyzs/Build/Products/Debug-iphonesimulator/AJFrameDevApp.app/PlugIns/AJFrameDevAppTests.xctest/AJFrameDevAppTests. One of the two will be used. Which one is undefined.
 
```
 
 A:问题的原因是AJFrameDevApp和AJFrameDevAppTests这两个target都链接了一个相同的静态库。
 于是我将Podfile的内容由
 
```
target 'AJFrameDevApp' do
    pod 'AJFrame', :path => '../'
end

target 'AJFrameDevAppTests' do
    pod 'AJFrame', :path => '../'
end
```

改为：

```
target 'AJFrameDevApp' do
    pod 'AJFrame', :path => '../'
    
    target 'AJFrameDevAppTests' do
        inherit! :search_paths
        
    end
end
```

Reference:http://stackoverflow.com/questions/30581884/class-is-implemented-in-both-one-of-the-two-will-be-used/30582486

### 6. 如何调用私有类的私有方法？真实的场景是开发了一个库，库中A持有一个私有类的属性，那么如何对私有类进行相应的单元测试呢？
A：Objective-C的runtime可以帮我们做到，

```
#include <objc/message.h>

NSString *originalString = @"testEncryptMethod";
    
NSData *originalData = [originalString dataUsingEncoding:NSUTF8StringEncoding];
    
AJBinaryChannel *binaryChannel = [AJBinaryChannel new];
binaryChannel.encryptKey = @"xorkey";
    
id wrapperEncryptor = (id)(binaryChannel.encryptor);
    
typedef NSData* (*send_type)(id, SEL, id);
    
send_type func = (send_type)objc_msgSend;
    
NSData *encryptData = func(wrapperEncryptor, @selector(encrypt:), originalData);
    
NSString *encryptString = [[NSString alloc] initWithData:encryptData encoding:NSUTF8StringEncoding];

XCTAssertNotEqualObjects(originalString, encryptString);
    
NSData *decryptData = func(wrapperEncryptor, @selector(decrypt:), encryptData);
    
NSString *decryptString = [[NSString alloc] initWithData:decryptData encoding:NSUTF8StringEncoding];

XCTAssertEqualObjects(originalString, decryptString);
```

### 7.Too many arguments to function call, expected 0, have 3
A:The answer is you need to strong type objc_msgSend for the compiler to build it:

```
typedef void (*send_type)(void*, SEL, void*);
send_type func = (send_type)objc_msgSend;
func(anItem.callback_object, NSSelectorFromString(anItem.selector), dict);
```

Reference:http://stackoverflow.com/questions/24922913/too-many-arguments-to-function-call-expected-0-have-3

### 8. Why do we need Base64?
A:Base64 is a group of similar binary-to-text encoding schemes that represent binary data in an ASCII string format by translating it into a radix-64 representation. 
A binary-to-text encoding is encoding of data in plain text. More precisely, it is an encoding of binary data in a sequence of characters. **These encodings are necessary for transmission of data when the channel does not allow binary data (such as email or NNTP) or is not 8-bit clean.**

Reference:https://en.wikipedia.org/wiki/Base64
https://en.wikipedia.org/wiki/Binary-to-text_encoding

