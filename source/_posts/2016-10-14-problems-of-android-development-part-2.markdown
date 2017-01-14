---
layout: post
title: "Android开发问题汇总(二)"
date: 2016-10-14 08:59:43 +0800
comments: true
categories: [Archives, Android] 
keywords: Android, development problems 
discription: 
---

### 1.Error running app: Default Activity not found
A: 问题的原因是 AndroidManifest.xml 文件配置的有问题。正确的样例如下：

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
package="edu.self.buildexample">

<application
android:allowBackup="true"
android:icon="@mipmap/ic_launcher"
android:label="@string/app_name"
android:supportsRtl="true"
android:theme="@style/AppTheme">

<activity android:name=".BuildExampleActivity">
<intent-filter>
<action android:name="android.intent.action.MAIN" />

<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
</activity>
</application>
</manifest>
```

### 2. How to rename java package in Android Studio?
A:[Android Studio Rename Package](http://stackoverflow.com/questions/16804093/android-studio-rename-package)  

### 3.No service of type Factory available in ProjectScopeServices
A:Change maven gradle plugin version to 1.4.1 in project build.gradle file

```
dependencies {
    classpath 'com.android.tools.build:gradle:2.2.2'
        classpath 'com.github.dcendents:android-maven-gradle-plugin:1.4.1'
}
```
Reference[No service of type Factory available in ProjectScopeServices](http://stackoverflow.com/questions/38825451/no-service-of-type-factory-available-in-projectscopeservices)

###4.Android Gradle cannot find symbol class Gson
A:In my case, I just added this line:

```
compile 'com.google.code.gson:gson:2.7'
```

on my app build.gradle file.

By now 2.7 is last current available version according to: https://mvnrepository.com/artifact/com.google.code.gson/gson

Please check this repository to be sure you are using last available version.

Reference:[Android Gradle cannot find symbol class Gson](http://stackoverflow.com/questions/17913704/android-gradle-cannot-find-symbol-class-gson)

<!--more-->

###5.Gradle sync failed: The SDK Build Tools revision (19.0.0) is too low for project ':app'. Minimum required is 19.1.0
A:设置更新版本的Build Tools: 

```
// Step 1
File > Project Structure ... > Modules(Select your module, eg:app) > Properties > Build Tools Version > 25.0.0

// Step 2:
Module build.gradle

android {
    ...
    buildToolsVersion "25.0.0"
    ...
} 
```

###6.Gradle sync failed: Could not find method android() for arguments [build_5v1bwdgcgpex8ek4ayinac3qa$_run_closure3@6518ea53] on root project 'ActionBarDemo' of type org.gradle.api.Project.
A:Root project gradle hasn't method android(), so remove it and setting in module gradle.

###7.Failed to resolve: junit:junit:4.12
A:It happens HTTPS is blocked by proxy, so we have to disable proxy.

```
// Step 1
Android Studio > Preferences... > System Settings > HTTP Proxy > No Proxy

// Step 2: Uncomment http proxy settings in gradle.properties
#systemProp.http.proxyHost=mirrors.neusoft.edu.cn
#systemProp.http.proxyPort=80

```
Reference:[Failed to resolve: junit:junit:4.12](http://stackoverflow.com/questions/32519219/error23-17-failed-to-resolve-junitjunit4-12)

###7.Error:Please select Android SDK
A:应用的Run 标红，打开弹出的对话底部提示error please select Android SDK.工程是从 eclipse 迁移过来的，target SDK 一开始是没安装的，安装之后标红依然没有消失。File > Project Structure... > Properties > Compile SDK Version 选择一个之前安装好的版本，工程构建之后标红消息，之后再选择刚安装的 target SDK 作为Compile SDK Version, 工程自动重新构建成功，标红消息。应该是安装 SDK
版本之后没有立即生效，连带 android.app 包都找不到。

###8.unknown attr app:popuptheme
A:The reason is miss relate name space:xmlns:app="http://schemas.android.com/apk/res-auto"

```
// Before
<android.support.v7.widget.Toolbar
android:id="@+id/my_toolbar"
android:layout_width="match_parent"
android:layout_height="?attr/actionBarSize"
android:background="?attr/colorPrimary"
android:elevation="4dp"
android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>

// After
<android.support.v7.widget.Toolbar
xmlns:app="http://schemas.android.com/apk/res-auto"
android:id="@+id/my_toolbar"
android:layout_width="match_parent"
android:layout_height="?attr/actionBarSize"
android:background="?attr/colorPrimary"
android:elevation="4dp"
android:theme="@style/ThemeOverlay.AppCompat.ActionBar"
app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>

```
###9.How to get Activity's content view?
A:

```
this.getWindow().getDecorView().findViewById(android.R.id.content)
or

this.findViewById(android.R.id.content)
or

this.findViewById(android.R.id.content).getRootView()
```

Reference:[How to get Activity's content view?](http://stackoverflow.com/questions/5273436/how-to-get-activitys-content-view)

###10.Conversion to Dalvik format failed with error 1

```
Dx 
UNEXPECTED TOP-LEVEL EXCEPTION:
    java.lang.RuntimeException: Exception parsing classes
    at com.android.dx.command.dexer.Main.processClass(Main.java:752)
    at com.android.dx.command.dexer.Main.processFileBytes(Main.java:718)
    at com.android.dx.command.dexer.Main.access$1200(Main.java:85)
    at com.android.dx.command.dexer.Main$FileBytesConsumer.processFileBytes(Main.java:1645)
    at com.android.dx.cf.direct.ClassPathOpener.processOne(ClassPathOpener.java:170)
    at com.android.dx.cf.direct.ClassPathOpener.processDirectory(ClassPathOpener.java:229)
    at com.android.dx.cf.direct.ClassPathOpener.processOne(ClassPathOpener.java:158)
    at com.android.dx.cf.direct.ClassPathOpener.processDirectory(ClassPathOpener.java:229)
    at com.android.dx.cf.direct.ClassPathOpener.processOne(ClassPathOpener.java:158)
    at com.android.dx.cf.direct.ClassPathOpener.processDirectory(ClassPathOpener.java:229)
    at com.android.dx.cf.direct.ClassPathOpener.processOne(ClassPathOpener.java:158)
    at com.android.dx.cf.direct.ClassPathOpener.processDirectory(ClassPathOpener.java:229)
    at com.android.dx.cf.direct.ClassPathOpener.processOne(ClassPathOpener.java:158)
    at com.android.dx.cf.direct.ClassPathOpener.processDirectory(ClassPathOpener.java:229)
    at com.android.dx.cf.direct.ClassPathOpener.processOne(ClassPathOpener.java:158)
    at com.android.dx.cf.direct.ClassPathOpener.processDirectory(ClassPathOpener.java:229)
    at com.android.dx.cf.direct.ClassPathOpener.processOne(ClassPathOpener.java:158)
at com.android.dx.cf.direct.ClassPathOpener.process(ClassPathOpener.java:144)
...
```
A:This android project dependent on serveral java projects, when I change these java project's jre from 1.8 to 1.6, it works. The deep reason isn't clear.

