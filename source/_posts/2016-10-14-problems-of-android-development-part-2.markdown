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
