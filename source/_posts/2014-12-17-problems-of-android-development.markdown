---
layout: post
title: "Android开发问题汇总"
date: 2014-12-17 09:38:20 +0800
comments: true
categories: [Archives, Android]
keywords: Android, 问题
discription: Android开发问题汇总
---
###Gradle使用代理服务器

```
// /Users/<username>/.gradle/gradle.properties (Mac)
# Http Proxy
systemProp.http.proxyHost=www.somehost.org
systemProp.http.proxyPort=8080
systemProp.http.proxyUser=userid
systemProp.http.proxyPassword=password
systemProp.http.nonProxyHosts=*.nonproxyrepos.com|localhost
```
Reference:[Gradle使用代理服务器](https://www.leimingshan.com/gradle/)

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

<!--more-->

###Error:Error retrieving parent for item: No resource found that matches the given name 'android:Theme.AppCompat.Light'.


###Project'X' is missing required Java project 'Y'
Solution: Open Java project 'Y'

###Unbound classpath container: 'Default System Library'
Solution: Eclipse > Preferences > Java > Installed JREs > Add > Standar VM > Select JRE Directory

My Mac JRE Directory is here:/Library/Java/JavaVirtualMachines/jdk1.8.0_11.jdk/Contents/Home/

### Eclipse Can't detect Android Device
Solution: Disable device's usb debuggin then enable, It then prompted me for authorization for that computer, select ok, and then it worked fine.

Reference:http://stackoverflow.com/questions/8799982/why-eclipse-can-not-detect-android-device

###Unable to resolve target "Android-5"
Solution:Android SDK Manager > Android 1.6 (API 4) > Intall

Eclipse > Preferences... > Android > Apply > OK

Project > Properties > Android > Project Build Target

### error: Error retrieving parent for item: No resource found that matches the given name '@android:TextAppearance.Material.SearchResult.Subtitle'.
Solution:Project > Properties > Android > Project Build Target > 21

Reference:http://stackoverflow.com/questions/26457096/appcompat-v7-r21-returning-error-in-values-xml

###Android studio Offline document path

Solution:file:///Users/dongmeiliang/Library/Android/sdk/docs/index.html

Reference:http://stackoverflow.com/questions/4974309/android-sdk-and-developer-guide-offline-or-pdf

###Error:Execution failed for task ':app:processDebugManifest'. Manifest merger failed : uses-sdk element cannot have a "tools:node" attribute

Solution:Add this line to uses-sdk tag like this:

```
<uses-sdk
    tools:node="merge"   <----This line do the magic
    android:minSdkVersion="14"
    android:targetSdkVersion="19" />
```

And add the tools name space in manifest :

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools" .....
.../>
```

Reference:http://stackoverflow.com/questions/26688711/uses-sdk-element-cannot-have-a-toolsnode-attribute

###Gradle DSL method not found: 'runProguard'

```
Error:(27, 0) Gradle DSL method not found: 'runProguard()'
**Possible causes:
The project 'Atomic4Mobile' may be using a version of Gradle that does not contain the method.
**Gradle settings**
The build file may be missing a Gradle plugin.
**Apply Gradle plugin**
```

Solution:runProguard was renamed to minifyEnabled in version 0.14.0.

Reference:http://stackoverflow.com/questions/27078075/gradle-dsl-method-not-found-runproguard

###error “Library projects cannot set applicationId”

```
Error: Library projects cannot set applicationId. applicationId is set to 'com.super.app' in default config.
```
Solution:Removing applicationId variable from the library's build.gradle file should resolve the issue.

Reference:http://stackoverflow.com/questions/27374933/android-studio-1-0-and-error-library-projects-cannot-set-applicationid

### How to increase the font size in Android Studio?

Solution: In mac book ,you can use two fingers to zoom in(increase font size) or zoom out for decrease font size, like when we zoomed image in mobilephone.

Reference:http://stackoverflow.com/questions/16590216/how-to-increase-the-font-size-in-android-studio

