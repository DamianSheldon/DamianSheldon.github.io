---
layout: post
title: "Android开发问题汇总"
date: 2014-12-17 09:38:20 +0800
comments: true
categories: [Archives, Android]
keywords: Android, 问题
discription: Android开发问题汇总
---

###Android Studio SDK Components安装提示操作超时

```
// MacOS
$ vim /etc/hosts
203.208.46.146 dl.google.com
203.208.46.146 dl-ssl.google.com
```

###Android Studio Check Updates提示连接出错

```
// MacOS
$ vim /Applications/Android\ Studio.app/Contents/Info.plist

// 搜索 VMOptions

// 添加如下内容
-Djava.net.preferIPv4Stack=true
-Didea.updates.url=http://dl.google.com/android/studio/patches/updates.xml
-Didea.patches.url=http://dl.google.com/android/studio/patches/

// 结果
<key>VMOptions</key>
<string>
-Dfile.encoding=UTF-8 -ea 
-Dsun.io.useCanonCaches=false 
-Djava.net.preferIPv4Stack=true
 -Djna.nosys=true 
 -Djna.boot.library.path=  
 -Djna.debug_load=true 
 -Djna.debug_load.jna=true 
 -Djsse.enableSNIExtension=false 
 -XX:+UseCodeCacheFlushing 
 -XX:+UseConcMarkSweepGC 
 -XX:SoftRefLRUPolicyMSPerMB=50 
 -Xverify:none -Xbootclasspath/a:../lib/boot.jar 
 -Djava.net.preferIPv4Stack=true 
 -Didea.updates.url=http://dl.google.com/android/studio/patches/updates.xml 
 -Didea.patches.url=http://dl.google.com/android/studio/patches/
 </string>

// 重启 Android Studio
```

###设置Gradle home

```
// Mac OS X
Use local gradle distribution > Gradle home > /Applications/Android Studio.app/Contents/plugins/gradle
```
###The project is using an unsupported version of Gradle

```
The project is using an unsupported version of Gradle.
Please point to a supported Gradle version in the project's Gradle settings or in the project's Gradle wrapper (if applicable.)

Consult IDE log for more details (Help | Show Log)
```