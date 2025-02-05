---
layout: post
title: "iOS App Distribution"
date: 2018-12-04 14:33:49 +0800
comments: true
categories: [Archives, iOS Development]
keywords: distribution, in-house, ad-hoc, testflight
description: Summary iOS App Distribution methods.
---

iOS App 有不少分发方法，她们散落在 Apple 的文档中，并未归总到一处，所以本文对她们进行了总结，方便查阅。  

首先我们简单梳理下这些分发方法，然后重点说明下 ipa 文件的安装方法。 Apple 制作的这张表格将分发方法归纳得很全：

| Method | Description |
| ------ | :----------- |
| App Store | Distributes your app through the App Store, signed with an Apple Developer Program distribution provisioning profile. |
| Ad Hoc | Distributes your app to testers with registered devices, signed with an ad hoc provisioning profile.<br>The devices need to be registered in your developer account and are limited to 100 devices per product family per year. If you don’t want to use a portion of these development devices for testing, distribute your app using TestFlight instead.<br>If you are a member of the Apple Developer Enterprise Program, choose this option to test your app. Only members of the Apple Developer Program have access to App Store Connect and TestFlight. |
| Development | Distributes your app to testers with registered devices, signed with a development provisioning profile. |
| Enterprise | Distributes your app to users in your organization, signed with an Apple Developer Enterprise Program distribution provisioning profile. |

虽然分发方法有很多，但可以分为两类：一类是通过 App Store 分发；另一类则是在 App Store 外分发。通过 App Store 分发时的流程很统一，都是上传应用到 iTunes Connection，等待审核发布，所以没有什么好说的。但是在 App Store 外发布时选择就多样了，我们既可以走像 App Store 分发那样流程的 TestFlight，也可以导出 ipa 文件然后安装。  

下面我们介绍在 iPhone 上安装 ipa 的方法：  

##Install using iTunes

> iTunes 12.7 for Mac was released on Tuesday with a major change in the app. Apple has redesign iTunes so that it focuses on sales of music, movies, TV shows, audiobooks, and podcasts. It no longer has an App Store for buying apps for your iPhone or iPad. Therefore, you can no long install your iOS App (.ipa file) through iTunes any longer.

1.	Download the .ipa file after the build completes.
2.	Open iTunes, go to App library.
3.	Drag and drop the downloaded .ipa file into the App library.
4.	Connect your device to iTunes and go to your device apps.
5.	Click Install button of the app and click Sync button. 

<!--more-->
##Install using Apple Configurator 2

1.	Install Apple Configurator 2 on your Mac from the App Store.
2.	Connect your device to your Mac.
3.	Open Apple Configurator 2, select your device. If you device doesn’t appear here, please make sure that your device is successfully connected to your Mac.


##Install using Xcode

1.	Connect your device to your Mac.
2.	Open Xcode, go to Window > Devices .
3.	Then, the Devices screen will appear. Choose the device you want to install the app on.
4.	Drag and drop your .ipa file into the Installed Apps.


##Install using OTA Deployment

OTA (Over-The-Air) Deployment enables you to install your built apps (ad-hoc build) via HTTPS.

1.	Download the .ipa file after the build completes.
2.	Upload the .ipa file to the site you want.
3.	Create a .plist file for this built application. The .plist file should look like this:

{% codeblock %}

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>items</key>
    <array>
        <dict>
            <key>assets</key>
            <array>
                <dict>
                    <key>kind</key>
                    <string>software-package</string>
                    <key>url</key>
                    <string>https://www.anysite.com/application/your_app.ipa</string>
                </dict>
            </array>
            <key>metadata</key>
            <dict>
                <key>bundle-identifier</key>
                <string>com.example.helloworld</string>
                <key>bundle-version</key>
                <string>1.0.0</string>
                <key>kind</key>
                <string>software</string>
                <key>title</key>
                <string>HELLO</string>
            </dict>
        </dict>
    </array>
</dict>
</plist>

{% endcodeblock %}


4.	Upload the `.plist` file to the site you want. Make sure this file must be accessed via HTTPS protocol. 
5.	Create a webpage embedded the link to the uploaded `.plist` file using special `itms-services://` protocol. See blow example: 
{% codeblock %}

<a href="itms-services://?action=download-manifest&url=https://example.com/manifest.plist">Install App</a>
{% endcodeblock %}

使用 OTA 方式部署安装，需要自己架设 Web 服务器，目前市面上也有免费提供此安装服务的产品，如 [Fir.im](https://fir.im/)，[蒲公英](https://www.pgyer.com/)。  

如果我们希望自己架设 Web 服务器来提供 OTA 分发，则相应地需要做些配置工作：  

## HTTPS

保证 ipa 文件是通过 HTTPS 访问，所以网站必须是由 iOS 信任的证书签名的。如果是没有信任锚的自签名证书，并且不能被 iOS 设备验证，那么安装会失败。

## Set server MIME types
 
你也许需要配置你的 web 服务器以便清单文件和应用文件能正确传输。
 
For the server, add the MIME types to the web service’s MIME types settings:
 
{% codeblock %}
 
application/octet-stream ipa 
text/xml plist
{% endcodeblock %} 
For Microsoft’s Internet Information Server (IIS), use IIS Manager to add the MIME type in the Properties page of the server:  

{% codeblock %}

.ipa application/octet-stream 
.plist text/xml
{% endcodeblock %} 
 
同时，如果设备是连接到一个封闭的内部网络，我们必须保证如下访问：

##Network configuration requirements

* https://ax.init.itunes.apple.com: The device obtains the current file-size limit for downloading apps over the cellular network. If this website isn’t reachable, installation may fail. 
* https://ppq.apple.com: The device contacts this website to check the status of the distribution certificate used to sign the provisioning profile. 

#Reference  

* [Distribution methods](https://help.apple.com/xcode/mac/current/#/dev31de635e5)  

* [App Distribution Guide](https://web.archive.org/web/20171114184350/https://developer.apple.com/library/content/documentation/IDEs/Conceptual/AppDistributionGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40012582-CH1-SW1)  
* [Non-market App Distribution](https://docs.monaca.io/en/products_guide/monaca_ide/deploy/non_market_deploy/#install-using-ota-deployment)  
 
* [Distribute in-house apps from a web server](https://help.apple.com/deployment/ios/#/apda0e3426d7)  

