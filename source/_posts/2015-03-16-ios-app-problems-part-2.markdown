---
layout: post
title: "iOS App 开发问题汇总（二）"
date: 2015-03-16 16:14:50 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS App
discription: iOS App 开发问题汇总
---

###问题描述：Ad Hoc分发应用

导出Ad Hoc授权应用
Xcode > Product > Archive > Export

安装Ad Hoc分发的应用
Open iTunes > File > Add To Library... > select Ad Hoc provisioning profile and App.ipa > Sync

###问题描述：Xcode编译出现如下警告提示：

[WARN]warning: no rule to process file xxx.h’ of type sourcecode.c.h for architecture armv7

解决办法:这是因为检查到有.h文件在编译列表中了。所以只要在列表中去掉就可以了。

点击你的xcode项目文件，然后点击[Build Phases]，确保在[Compile Sources]中没有.h文件。

Reference:http://hearrain.com/2014/01/774

<!--more-->

###问题描述:ld: warning: directory not found for option '-L/Users/dongmeiliang/Documents/soh.client.iOS/sohiOSApp/sohiOSApp/Component/BaiduMap_IOSSDK_v2-2/Release-iphonesimulator'
Undefined symbols for architecture x86_64:
  "_DDExtractFileNameWithoutExtension", referenced from:
      ___57-[AJSocketBinaryClientChannel setupReadTimerWithTimeout:]_block_invoke76 in libPods-sohiOSApp-AJFrame.iOS.a(AJSocketBinaryClientChannel.o)
      -[AJSocketBinaryClientChannel doReadTimeout] in libPods-sohiOSApp-AJFrame.iOS.a(AJSocketBinaryClientChannel.o)
      -[AJSocketBinaryClientChannel endCurrentRead] in libPods-sohiOSApp-AJFrame.iOS.a(AJSocketBinaryClientChannel.o)
  "_OBJC_CLASS_$_DDLog", referenced from:
      objc-class-ref in libPods-sohiOSApp-AJFrame.iOS.a(AJSocketBinaryClientChannel.o)
  "_OBJC_CLASS_$_GCDAsyncSocket", referenced from:
      objc-class-ref in libPods-sohiOSApp-AJFrame.iOS.a(AJTcpConnector.o)
ld: symbol(s) not found for architecture x86_64

解决办法：从输出的日志可以看出问题是出在链接，由Cocoapods管理的第三方库安装后链接配置项没有设置正确，需要手动设置没有被自动链接的库，例如这里我需要链接：-l"Pods-CocoaAsyncSocket"。

###问题描述:dyld: could not load inserted library '/usr/lib/libgmalloc.dylib' because image not found

解决办法：Edit Scheme > Run > Diagnostics > Memory Managerment 

Malloc uncheck Enable Scribble  
		uncheck Enable Guard Edges  
		uncheck Enable Guard Malloc  
		
Reference:http://blog.csdn.net/chocolateloveme/article/details/18258443

###问题描述：APNs服务器地址

答案：
gateway.push.apple.com 2195 (Product)  
gateway.sandbox.push.apple.com 2195 (Development)

###问题描述：/Users/user/XCodeWork/iPhoneDev/MyAppSourceFolder/en.lproj/MainWindow-iPhone.xib: 
  Compilation failed. Unable to write to path:        
  /Users/user/Library/Developer/Xcode/DerivedData/MYAPPNAME-
  dudnhxzgpqtcnqcgaguirvkhmvco/Build/Intermediates/ArchiveIntermediates/
  MYAPPNAME/InstallationBuildProductsLocation/Applications/MyAppname.app/
  en.lproj/MainWindow-iPhone.nib
  
解决办法：For every xib's that we've problems, in the right panel go to the first tab, "Interface Builder Document" -> "Builds for" and select "iOS 7.0 and Later". My old configuration was "iOS 4.3 and Later".
  
Reference:http://stackoverflow.com/questions/19688276/interface-builder-xib-error-unable-to-write-to-path

###问题描述:duplicate symbol _OBJC_METACLASS_$_SDWebImagePrefetcher in:
    /Users/dongmeiliang/Library/Developer/Xcode/DerivedData/jiaxiaotong-dxrmnnwbexugoqfekesgvdiijcdm/Build/Products/Debug-iphonesimulator/libPods-weima-SDWebImage.a(SDWebImagePrefetcher.o)
    /Users/dongmeiliang/Library/Developer/Xcode/DerivedData/jiaxiaotong-dxrmnnwbexugoqfekesgvdiijcdm/Build/Products/Debug-iphonesimulator/libPods-SDWebImage.a(SDWebImagePrefetcher.o)
ld: 727 duplicate symbols for architecture x86_64

解决办法：问题出现的原因是我用CocoaPods管理第三方库，在Build Settings > Linking > Other Linker Flags 里面即显示指定了-l"Pods-SDWebImagePrefetcher",又设置了变量$(inherited)。我去掉$(inherited)就好。

###问题描述:2015-04-05 22:35:26.917 weima[67059:2338954] NSScanner: nil string argument
2015-04-05 22:35:26.917 weima[67059:2338954] NSScanner: nil string argument
libc++abi.dylib: terminate_handler unexpectedly threw an exception


###Cannot send push notifications using Javapns/Javaapns SSL handshake failure
解决办法：
```
developer_identity.cer <= download from Apple (aps cert)
mykey.p12 <= Your private key (CSR generate private key or install aps cert get a private key both ok)

openssl x509 -in developer_identity.cer -inform DER -out developer_identity.pem -outform PEM
openssl pkcs12 -nocerts -in mykey.p12 -out mykey.pem
openssl pkcs12 -export -inkey mykey.pem -in developer_identity.pem -out iphone_dev.p12
```
Reference:http://stackoverflow.com/questions/12585858/cannot-send-push-notifications-using-javapns-javaapns-ssl-handshake-failure

###导航栏如何显示两行文本？
解决办法：After setting up your titleView or titleLabel, call sizeToFit on it, also make sure titleLabel.textAlignment = UITextAlignmentCenter. It'll be centered in the width of the navbar rather than in the space between the button edge and far edge of navbar.

Reference http://stackoverflow.com/questions/3681990/adjusting-navigationitem-titleviews-frame

###SVN how to resolve “local unversioned, incoming add upon update”

```
$ svn status
D     C logs
      >   local unversioned, incoming add upon update
Summary of conflicts:
  Tree conflicts: 1
```

解决办法:

```
$ svn resolve --accept working logs
Resolved conflicted state of 'logs'
$ svn revert logs
Reverted 'logs'
$ svn status
```
Reference http://tomhennigan.blogspot.com/2012/01/resolve-tree-conflict-svn-local.html

###问题描述：Calling a C function from within a function in a .mm file

解决办法：The .mm file is looking to call a mangled version of the function name. You need either __BEGIN_DECLS and __END_DECLS around the C function declarations seen by the C++-compiled file, or you need to do the equivalent yourself. The idea is to mark those function declarations as extern "C" when seen by an (Obj-)C++ compiler, but not when seen by any other sort of compiler.

Referenc: http://stackoverflow.com/questions/4984523/calling-a-c-function-from-within-a-function-in-a-mm-file

###问题描述：Compilation failed. Unable to write to path

```
    Underlying Errors:
        Description: The file “objects.nib” doesn’t exist.
        Failure Reason: The file doesn’t exist.
        Underlying Errors:
            Description: The operation couldn’t be completed. No such file or directory
            Failure Reason: No such file or directory
        Description: “DMLUserElementCell~iphone.nib” couldn’t be removed.
        Failure Reason: The file doesn’t exist.
        Underlying Errors:
            Description: The operation couldn’t be completed. No such file or directory
            Failure Reason: No such file or directory
```

解决办法：在Target > Build Phases > Compile Sources里去掉出错的nib文件。

Reference:http://stackoverflow.com/questions/20570340/xcode-5-storyboard-compile-failure

###问题描述：Where iPhoto Pictures are Located and How to Access the iPhoto Library and Picture Files

解决办法：iPhoto 11 (9.0) Photo Library Storage Location:~/Pictures/iPhoto Library.photolibrary/Masters/

Reference:http://osxdaily.com/2011/08/30/where-iphoto-pictures-are-located/


