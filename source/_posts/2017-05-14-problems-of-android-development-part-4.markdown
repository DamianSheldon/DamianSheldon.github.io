---
layout: post
title: "Android 开发问题汇总(四)"
date: 2017-05-14 16:56:19 +0800
comments: true
categories: [Archives, Android] 
keywords: Android, development 
description: Problems of android development. 
---
###1.怎么给 Button 加上圆角?
A:Create an xml inside Drawable Folder with below code. Name it rounded_red_border.xml  

```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
 
// The width and color of the border   
 <stroke
        android:width="4dp"
        android:color="#de3d3d" />
 
// The desired corner radius. reduce it to keep it less rounded
    <corners android:radius="360dp" />
 
// Add your desired padding
    <padding
        android:left="20dp"
        android:top="10dp"
        android:right="20dp"
        android:bottom="10dp"    >
    </padding>
 
</shape>
```

Applying the border to a Layout or View  

Set the above drawable as a background to your Layout or View like LinearLayout, FrameLayout, TextView, Button etc.  

```
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
android:layout_width="match_parent"
android:layout_height="match_parent">
 
 
// Apply as the background of TextView
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="YOUR TEXT"
    android:textSize="20dp"
    android:textStyle="bold"
    android:background="@drawable/rounded_red_border"
    android:textColor="#000"
    android:layout_centerInParent="true"/>
 
 
</RelativeLayout>
```
<!--more-->
Reference:[Add Rounded Corners to Views and Layouts Android](http://www.gadgetsaint.com/tips/rounded-corners-views-layouts-android/#.WRgR3FOGORt)  

###2.Error:All flavors must now belong to a named flavor dimension.
A:在主app的build.gradle里面的

```
 defaultConfig {
 	targetSdkVersion：***
	minSdkVersion ：***
	versionCode：***
 	versionName ：***
	// 版本名后面添加一句话，意思就是flavor dimension
	// 它的维度就是该版本号，这样维度就是都是统一的了
	flavorDimensions "versionCode"
}
```
Reference:[解决Error:All flavors must now belong to a named flavor dimension.](https://blog.csdn.net/SYIF88/article/details/75009663)  

###3.error: resource android:attr/fontVariationSettings not found.
A:This is caused by an incompatibility with the android support library that changed to version 28 in the last day or so. Add follow contents to build.gradle:

```
configurations.all {
    resolutionStrategy {
        force 'com.android.support:support-v4:27.1.0'
    }
}
```
Reference:[Problem Isolated - Need Solution](https://github.com/crosswalk-project/cordova-plugin-crosswalk-webview/issues/205#issuecomment-371669478)  

###4.Application error. The connection to the server was unsuccessful
A:Add the next line into the config.xml in side the tag.

```
<preference name="loadUrlTimeoutValue" value="700000" />
```

Reference:[Application error. The connection to the server was unsuccessful](https://forum.ionicframework.com/t/application-error-the-connection-to-the-server-was-unsuccessful/67584/3)  

###5.How to get SQLite database from Android real device?
A:Device File Explorer can archive this goal. We can open it this way:

View -> Tool Windows -> Device File Explorer

Once you have Device File Explorer window open, use your mouse to navigate to the following path:

data -> data -> your.package.name -> databases

Inside the databases folder you should see the database you want to explore, do a right click and Save As... select your desired computer destination folder.

Reference:[Get SQLite database from Android app](https://stackoverflow.com/questions/21062187/get-sqlite-database-from-android-app)  

###6.AAPT: error: file failed to compile
A:一个旧的工程编译报错，错误和 nine-patch 图片资源相关，详细的错误为:AAPT: error: file failed to compile，搜索了一圈没找到有效的解决办法。既然错误是和 aapt 相关，于是尝试使用 Android Studio 的命令行工具 aapt 来输出更多错误信息。  

```
$ /Users/meiliang/Library/Android/sdk/build-tools/28.0.3/aapt2 compile ~/Documents/AndroidProjects/LocationAware/app/src/main/res/drawable-mdpi/button_on.9.png -o /tmp/compiled/ -v
/Users/meiliang/Documents/AndroidProjects/LocationAware/app/src/main/res/drawable-mdpi/button_on.9.png: note: compiling PNG.
error: found an invalid color.
```

得到了无效的颜色的错误信息，但还是不知道如何解决。只能继续啃 Android Studio 用户指南的[Create resizable bitmaps (9-Patch files)](https://developer.android.google.cn/studio/write/draw9patch?hl=en), 及从中引申出来的 [NinePatch drawables](https://developer.android.google.cn/guide/topics/graphics/drawables#nine-patch) ,看完之后对 nine-patch 有了进一步的认识，但是还是不知道如何创建，先尝试用 GIMP 来设置可以拉伸区域，失败了，于是上 Google 搜索相关视频教程，原来是拖动来设置拉伸区域而不是官方文档上面的点击，难怪点击一直没有效果，设置好拉伸区域后就能成功编译了。  

###7.Android 工程设置 `svn:global-ignores, svn:ignore`
A:公司是使用 svn 来管理源代码，所以需要为 android 工程设置 svn ignore 规则。 svn 主要通过 `svn:global-ignores, svn:ignore` 来设置忽略规则。global-ignores 从名称可以看出它是全局相关的，ignore 则是和特定代码库相关的。  

svn 的 ignore 规则是面向文件的，所以 global-ignores 可以忽略整个目录，但如果想忽略目录中部分文件则需要设置 ignore，例如可以配置 global-ignores 它可以忽略 `.idea` 目录，但如果想忽略 `.idea` 目录中部分文件则需要设置 ignore。个人觉得它不如 git ignore 方便。 下面我们参考 github 的 Android.gitignore 来为工程设置 svn global-ignores, ignore。  

主要是使用 `$svn propedit svn:global-ignores .` 和 `$ svn propedit svn:ignore .idea`。

```
$svn propedit svn:global-ignores .
# Built application files
*.apk
*.aar
*.ap_
*.aab

# Files for the ART/Dalvik VM
*.dex

# Java class files
*.dex

# Generated files
bin
gen
out
release

# Gradle files
.gradle
build

# Local configuration file (sdk path, etc)
local.properties

# Proguard folder generated by Eclipse
proguard

# Log Files
*.log

# Android Studio Navigation editor temp files
.navigation

# Android Studio captures folder
captures

# IntelliJ
*.iml

# Keystore files
# Uncomment the following lines if you do not want to check your keystore files in.
*.jks
*.keystore

# External native build folder generated in Android Studio 2.2 and later
.externalNativeBuild
.cxx

# Version control
vcs.xml

$ svn propedit svn:ignore .  
idea
workspace.xml
tasks.xml
gradle.xml
assetWizardSettings.xml
dictionaries
libraries
# Android Studio 3 in .gitignore file.
caches
modules.xml
# Comment next line if keeping position of elements in Navigation Editor is relevant for you
navEditor.xml
```

* Reference: [Ignoring Unversioned Items](http://svnbook.red-bean.com/en/1.8/svn.advanced.props.special.ignore.html)  

###8.`.aidl` 文件的目标位置
A:当我们使用 AIDL 调用方法时需要包含 `.aidl` 文件，官方文档上是说 `Include the .aidl file in the project src/ directory.` 我实际测试下来发现要包含在 `src/main/` 目录下才会生效。 

```
$ls app/src/main/
AndroidManifest.xml        assets/                    ic_launcher-playstore.png  libs/
aidl/                      com/                       java/                      res/
```

###9.Activity did not call finish() prior to onResume() completing
A:It was workaround by using Theme.Translucent instead of Theme.NoDisplay in the manifest.

Reference:  

* [Activity did not call finish? (API 23)](https://stackoverflow.com/questions/32169303/activity-did-not-call-finish-api-23) 
* [Activity crash with @android:style/Theme.NoDisplay](https://web.archive.org/web/20151116170752/https://code.google.com/p/android-developer-preview/issues/detail?id=2353)  

###10.DownLoadManager failed with ERROR_UNKOWN
A:First check your url, http schema is disable by default on API level 27 or higher. If this is your case, you can change to https schema or enable http via [network security configuration](https://developer.android.google.cn/training/articles/security-config?hl=en#CleartextTrafficPermitted).  

Second, you can find clue in logcat, remind change it to No Filter, search with DownLoadManager, I find file permission exception from here.

###11.Logcat to file
A:

```
$adb shell
$logcat -b main,system,crash -f /mnt/sdcard/logs/logcat.log -r 32 -n 65535 &
```

###12.将 ionic 的 android 工程单独提交到 svn 仓库
A:

{% codeblock %}
$ mkdir RAWApp
$ svn import RAWApp http://url/svn/jwt_v3/ydjwv3/trunk/working/projects/client/cssj/RAWApp 
$ rm -rf RAWApp
$ svn checkout http://url/svn/jwt_v3/ydjwv3/trunk/working/projects/client/cssj/RAWApp
$ rsync -avuz platforms/android/ RAWApp
$ cd RAWApp
$ svn add --force .
$ svn commit
{% endcodeblock %}


