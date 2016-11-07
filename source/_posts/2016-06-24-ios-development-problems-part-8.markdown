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

### 9. 360° panorama libraries for ios
A:[panoramagl](https://code.google.com/archive/p/panoramagl/)

Reference:http://stackoverflow.com/questions/3763978/360-panorama-libraries-for-ios

### 10. When should we use macro in iOS?
A: As we known, macro has many disatanges:1.lack type info;2.can be redfine. Since its a keyword in C, when should we use it?

There are occasions where macros provide functionality not available through other means.

For example, I have a simple macro I sometimes use when debugging Objective-C to log when certain methods are called. This can be done like so:

`NSLog(@"%@: %@", NSStringFromClass([self class]), NSStringFromSelector(_cmd));`
This can’t be moved into an Objective-C method because it will always log the name of that method. (Obviously it can’t be moved into a C method, as there is no self and no _cmd variables available.) Creating a macro for this is straightforward, however:

`#define LOG_SELECTOR()  NSLog(@"%@: %@", NSStringFromClass([self class]), NSStringFromSelector(_cmd));`

Reference:[Appropriate Use of C Macros for Objective-C Developers](http://weblog.highorderbit.com/post/11656225202/appropriate-use-of-c-macros-for-objective-c)  

### 11.Open Source VoIP/SIP Objective-C Code?
A:siphon (http://code.google.com/p/siphon/).

>Home of the World's first free SIP/VoIP application for iPhone and iPod Touch 1 and 2.

>Siphon SIP/VoIP project is the first in his category that works on iPhone and iPod Touch 2 with headset for all SIP providers. It is a nat>ive application approved running on 2.X using internal micro/speaker and headset.

>The Application supports the SIP standard, preserving compatibility with hundreds of SIP providers and offers a GUI which preserves the ap>>ple design of native iPhone applications.

Reference:[iOS: Open Source VoIP/SIP Objective-C Code](http://stackoverflow.com/questions/1493050/ios-open-source-voip-sip-objective-c-code)

### 12. How to show a search icon in indexed table view?
A:UITableViewIndexSearch.

>If the data source includes this constant string in the array of strings it returns in sectionIndexTitlesForTableView:, the section index displays a magnifying glass icon at the corresponding index location. This location should generally be the first title in the index.

```
- (NSArray *)sectionIndexTitles
{
    if (!_sectionIndexTitles) {
        NSArray *titles = [UILocalizedIndexedCollation currentCollation].sectionIndexTitles;

        NSMutableArray *tmpArray = [NSMutableArray arrayWithCapacity:titles.count + 1];
        [tmpArray addObject:UITableViewIndexSearch];

        for (NSString *title in titles) {
            [tmpArray addObject:title];
        }

        _sectionIndexTitles = tmpArray.copy;
    }
    return _sectionIndexTitles;
}

- (NSArray *)sectionIndexTitlesForTableView:(UITableView *)tableView {
        return self.sectionIndexTitles;
}
```
How to implement a function that active search bar when user tap magnifying glass icon?

First, add support for indexed table view;

```
// Create the search results controller and store a reference to it.
MySearchResultsController* resultsController = [[MySearchResultsController alloc] init];
self.searchController = [[UISearchController alloc] initWithSearchResultsController:resultsController];
 
 // Use the current view controller to update the search results.
 self.searchController.searchResultsUpdater = self;
  
  // Install the search bar as the table header.
  self.tableView.tableHeaderView = self.searchController.searchBar;
   
   // It is usually good to set the presentation context.
   self.definesPresentationContext = YES;
```

Note: set navigation bar to translucent via Interface Builder or Programatically `    self.navigationController.navigationBar.translucent = YES;`, otherwise search bar will animate out table view.

Reference:[UISearchController searchBar in tableHeaderView animating out of the screen](http://stackoverflow.com/questions/26222671/uisearchcontroller-searchbar-in-tableheaderview-animating-out-of-the-screen)  

Second, hack tableView:sectionForSectionIndexTitle:atIndex: method.

```
- (NSInteger)tableView:(UITableView *)tableView sectionForSectionIndexTitle:(NSString *)title atIndex:(NSInteger)index {
    if (!index) {
        self.searchController.active = YES;
    }
    return [self.collation sectionForSectionIndexTitleAtIndex:index];
}
```

Reference:[Making UITableViewIndexSearch jump to tableHeaderView instead of section 0](http://stackoverflow.com/questions/26149526/making-uitableviewindexsearch-jump-to-tableheaderview-instead-of-section-0)  

### 13.Transparent UIToolbar
A:

```
[self.toolbar setBackgroundImage:[UIImage new]
forToolbarPosition:UIBarPositionAny
barMetrics:UIBarMetricsDefault];
[self.toolbar setShadowImage:[UIImage new]
forToolbarPosition:UIBarPositionAny];
```

Reference:[How to draw a transparent UIToolbar or UINavigationBar in iOS7](http://stackoverflow.com/questions/18969248/how-to-draw-a-transparent-uitoolbar-or-uinavigationbar-in-ios7)

### 14.App Transport Security Dictionary Details
A:Information Property List Key Reference.
