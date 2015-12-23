---
layout: post
title: "iOS App 开发问题汇总（四）"
date: 2015-10-15 15:04:13 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS Development
discription: iOS App Development
---
### 1. Suppressing deprecated warnings
```
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Wdeprecated-declarations"
[reciver deprecatedMethod];
#pragma clang diagnostic pop
```

### 2. Warning libopencore-amrnb.a, missing required architecture arm64
```
ld: warning: ignoring file /Users/dongmeiliang/Downloads/VoiceConvert/VoiceConvert/VoiceConvert/lib/libopencore-amrnb.a, missing required architecture arm64 in file /Users/dongmeiliang/Downloads/VoiceConvert/VoiceConvert/VoiceConvert/lib/libopencore-amrnb.a (3 slices)
ld: warning: ignoring file /Users/dongmeiliang/Downloads/VoiceConvert/VoiceConvert/VoiceConvert/lib/libopencore-amrwb.a, missing required architecture arm64 in file /Users/dongmeiliang/Downloads/VoiceConvert/VoiceConvert/VoiceConvert/lib/libopencore-amrwb.a (3 slices)
Undefined symbols for architecture arm64:
  "_Decoder_Interface_init", referenced from:
      DecodeAMRFileToWAVEFile(char const*, char const*) in libVoiceConvert.a(amrFileCodec.o)
  "_Decoder_Interface_Decode", referenced from:
      DecodeAMRFileToWAVEFile(char const*, char const*) in libVoiceConvert.a(amrFileCodec.o)
  "_Decoder_Interface_exit", referenced from:
      DecodeAMRFileToWAVEFile(char const*, char const*) in libVoiceConvert.a(amrFileCodec.o)
  "_Encoder_Interface_init", referenced from:
      EncodeWAVEFileToAMRFile(char const*, char const*, int, int) in libVoiceConvert.a(amrFileCodec.o)
  "___gxx_personality_v0", referenced from:
      +[VoiceConverter amrToWav:wavSavePath:] in libVoiceConvert.a(VoiceConverter.o)
      +[VoiceConverter wavToAmr:amrSavePath:] in libVoiceConvert.a(VoiceConverter.o)
  "_Encoder_Interface_Encode", referenced from:
      EncodeWAVEFileToAMRFile(char const*, char const*, int, int) in libVoiceConvert.a(amrFileCodec.o)
  "_Encoder_Interface_exit", referenced from:
      EncodeWAVEFileToAMRFile(char const*, char const*, int, int) in libVoiceConvert.a(amrFileCodec.o)
```
Solution: Recompile a static library refer [opencore-amr-iOS](https://github.com/feuvan/opencore-amr-iOS)
<!-- more -->
### 3. Undefined symbols for architecture arm64:
  "___gxx_personality_v0", referenced from:
      +[VoiceConverter amrToWav:wavSavePath:] in libVoiceConvert.a(VoiceConverter.o)
      +[VoiceConverter wavToAmr:amrSavePath:] in libVoiceConvert.a(VoiceConverter.o)
ld: symbol(s) not found for architecture arm64

Solution: Project > Targets > Select your product target > Build Settings > Other Linker Flags > Add -l"stdc++"

Reference:http://stackoverflow.com/questions/2846679/creating-an-objective-c-static-library-in-xcode

### 4. Suppressing performSelector may cause leaks warnings

```
#pragma clang diagnostic push
#pragma clang diagnostic ignored "-Warc-performSelector-leaks"
    [self.ticketTarget performSelector: self.ticketAction withObject: self];
#pragma clang diagnostic pop
```

Reference:http://stackoverflow.com/questions/7017281/performselector-may-cause-a-leak-because-its-selector-is-unknown

###4.How to remove UINavigationBar and UISearchBar hairline?
Solution:  

> For a custom shadow image to be shown, a custom background image must also be set with the setBackgroundImage:forBarMetrics: method. If the default background image is used, then the default shadow image will be used regardless of the value of this property.

So we can cut a 1x1 pixel pure color image, set this image as navigation bar's shadow image.

Reference:http://stackoverflow.com/questions/19037464/how-to-remove-uinavigationbar-and-uisearchbar-hairline  
http://stackoverflow.com/questions/19226965/how-to-hide-ios7-uinavigationbar-1px-bottom-line

### 5. Tab Bar item 设置的默认图片是白色的，但是显示却是灰色  
Solution:

```
UIImage *userImage = [UIImage imageNamed:@"User"];
userImage = [userImage imageWithRenderingMode:UIImageRenderingModeAlwaysOriginal];
UITabBarItem *friends = [[UITabBarItem alloc] initWithTitle:@"Recommand Friends" image:userImage selectedImage:userImage];
```

### 6. UIWebView's cache doesn't work
Solution:
> Create requestObj with specific cache policy ReturnCacheDataElseLoad if you can tell there is no network, or use the default behavior.

Reference:http://stackoverflow.com/questions/32605084/uiwebview-cache-not-working

###7.How to only show bottom border of UITextField
Solution:

```
var bottomLine = CALayer()
bottomLine.frame = CGRectMake(0.0, myTextField.frame.height - 1, myTextField.frame.width, 1.0)
bottomLine.backgroundColor = UIColor.whiteColor().CGColor
myTextField.borderStyle = UITextBorderStyle.None
myTextField.layer.addSublayer(bottomLine)
```

Reference:http://stackoverflow.com/questions/31107994/how-to-only-show-bottom-border-of-uitextfield-in-swift

### 7. 让Button的大小比它的内容略大
Solution:

```
_loginViaSMSButton.contentEdgeInsets = UIEdgeInsetsMake(0, 20, 0, 20);

```

### 8. UITableViewStyleGrouped style UITableView has extra space at top

Solution:

> UITableViewStyleGrouped divides each section into a "group" by inserting that extra padding…similar to what it did pre-iOS7, but clear instead of colored by default, and just at the top instead of all the way around. If you don't want the padding by default, use UITableViewStylePlain.

Otherwise, if you need to keep the style the same, do what this other posted from that link recommended and change the content inset:

```
self.tableView.contentInset = UIEdgeInsetsMake(-36, 0, 0, 0);

```
Reference:http://stackoverflow.com/questions/20305943/why-extra-space-is-at-top-of-uitableview-simple
