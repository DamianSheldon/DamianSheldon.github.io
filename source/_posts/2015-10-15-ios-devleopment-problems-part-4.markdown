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

### 7.How to only show bottom border of UITextField
Solution:

```
var bottomLine = CALayer()
bottomLine.frame = CGRectMake(0.0, myTextField.frame.height - 1, myTextField.frame.width, 1.0)
bottomLine.backgroundColor = UIColor.whiteColor().CGColor
myTextField.borderStyle = UITextBorderStyle.None
myTextField.layer.addSublayer(bottomLine)
```

Reference:http://stackoverflow.com/questions/31107994/how-to-only-show-bottom-border-of-uitextfield-in-swift

### 8. 让Button的大小比它的内容略大
Solution:

```
_loginViaSMSButton.contentEdgeInsets = UIEdgeInsetsMake(0, 20, 0, 20);

```

### 9. UITableViewStyleGrouped style UITableView has extra space at top

Solution:

> UITableViewStyleGrouped divides each section into a "group" by inserting that extra padding…similar to what it did pre-iOS7, but clear instead of colored by default, and just at the top instead of all the way around. If you don't want the padding by default, use UITableViewStylePlain.

Otherwise, if you need to keep the style the same, do what this other posted from that link recommended and change the content inset:

```
self.tableView.contentInset = UIEdgeInsetsMake(-36, 0, 0, 0);

```
Reference:http://stackoverflow.com/questions/20305943/why-extra-space-is-at-top-of-uitableview-simple

### 10. 使用 UISearchDisplayController 遇到的问题

使用代码创建的 UISearchDisplayController 必须用一个属性来持有它，个人觉得这里有点奇怪，因为 UIViewController 在头文件中的声明是：

```
@property(nonatomic, readonly, strong) UISearchDisplayController *searchDisplayController
```

理论上来讲既然是 strong 那么这个对象应该会存在才对，但不知道什么原因，这样并不行，它会在内存中被销毁，还是得在扩展里用一个属性来持有它。

```
*.m

@property (nonatomic) UISearchDisplayController *wt_searchDisplayController;

self.wt_searchDisplayController = [[UISearchDisplayController alloc] initWithSearchBar:self.searchBar contentsController:self];
self.wt_searchDisplayController.delegate = self;
self.wt_searchDisplayController.searchResultsDataSource = self.searchCoordinator;
self.wt_searchDisplayController.searchResultsDelegate = self.searchCoordinator;
[self.wt_searchDisplayController.searchResultsTableView registerClass:[RCConversationCell class] forCellReuseIdentifier:sSearchCellIdentifier];

```

问题还没有完全解决，我应用的导航结构是 UINavigationController > UITabBarController [VC1, VC 2, VC3], 在 VC1 界面里需要搜索功能，然后我在 VC1 navigation bar 下方放置了一个 search bar， 这个 search bar 和 searchDisplayController 关联起来，结果却出现 search result tableView 覆盖 search bar 的情况，最后的解决办法是调整导航结构 UINavigationController > UITabBarController [(UINavigationController > VC1), (UINavigationController > VC2), (UINavigationController > VC3)], 也就是先把 UIViewController 嵌入到 UINavigationController 中， 再把它做为 UITabBarController 的 childController.

### 11. Controller 的视图 Layer 上添加 AVCaptureVideoPreviewLayer， 然后再在 Controller 的 View 上添加一个扫描框视图，用系统的 View 作为扫描框视图时能正常显示相机的捕获的内容， 用自定义的视图时，扫描框内全是黑的。

Solution:在自定义视图的初始化方法中设置 opaque 为 NO。

### 12. You are not authorized to use this service.
Solution: I recently enable Apple ID two-step verification. When I upload app to App Store via Xcode it inform me "You are not authorized to use this service." I feel very strange. I decide to use Application Loader after search with google.  Sign in Application Loader need app-specific password, you can get it follow instructs: 

1. Sign in to your [Apple ID account page](https://appleid.apple.com/account/home);
2. In the Security section, click Edit.
3. Click Generate Password and follow the steps on your screen.

Reference:https://support.apple.com/en-us/HT204397

Now we can upload app with Application Loader:
1. Xcode > Open Developer Tool > Application Loader
2. Sign in Application Loader with app-specific password
3. Click on "Choose"
4. Pick the IPA you have just exported
5. Follow submission process

Reference:http://stackoverflow.com/questions/33291394/xcode-7-1-not-authorized-to-use-this-service-error

### 13.

```
ERROR ITMS-90535: "Unexpected CFBundleExecutable Key. The bundle at 'Payload/sohiOSApp.app/TencentOpenApi_IOS_Bundle.bundle' does not contain a bundle executable. If this bundle intentionally does not contain an executable, consider removing the CFBundleExecutable key from its Info.plist and using a CFBundlePackageType of BNDL. If this bundle is part of a third-party framework, consider contacting the developer of the framework for an update to address this issue."
```

Solotion:Removing the CFBundleExecutable key from TencentOpenApi_IOS_Bundle.bundle's Info.plist and using a CFBundlePackageType of BNDL.



