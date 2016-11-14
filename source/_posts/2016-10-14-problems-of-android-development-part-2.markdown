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

<!--more->

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
