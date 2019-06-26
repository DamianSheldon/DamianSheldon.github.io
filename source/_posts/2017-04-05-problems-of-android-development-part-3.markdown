---
layout: post
title: "Android开发问题汇总(三)"
date: 2017-04-05 08:48:36 +0800
comments: true
categories: [Archives, android] 
keywords: Android, Problems 
description: 
---

###1. How to define custom attributes?
A:Currently the best documentation is the source. You can take a look at it [here(arrts.xml)](https://github.com/android/platform_frameworks_base/blob/master/core/res/res/values/attrs.xml).  

You can define attributes in the top `<resources>` element or inside of a `<declare-styleable>` element. If I'm going to use an attr in more than on place I put it in the root element. Note , all attributes share the same global namespace. That means that even if you create a new attribute inside of a `<declare-styleable>` element it can be used outside of it and you cannot create another attribute with the same name of a different type.

An `<attr>` element has two xml attributes `name` and `format`. `name` lets you call it something and this how you end up refering to it in code, e.g., R.attr.my_attribute. The `format` attribute can have different values depending on the type of attribute you want.  

* reference - if it references another resource id(e.g, "@color/my_color", "@layout/my_layout")
* color
* boolean
* dimension
* float
* integer
* string
* fraction
* enum - normally implicitly defined
* flag - normally implicitly defined

You can set the format to multiple types by using |, e.g., `format="reference|color"`.

enum attributes can be defined as follows:

```
<attr name="my_enum_attr"> 
    <enmu name="value1" value="1" />
    <enmu name="value2" value="2" />
</attr>
```

flag attributes are similar except the values need to defined so they can be bit ored together:

```
<attr name="my_flag_attr">
    <flag name="fuzzy" value="0x01" />
    <flag name="cold" value="0x02" />
</attr>
```
<!--more-->
In addition to attributes there is the `<declare-styleable>` element. This allows you to define attributes a custom view can use. You do this by specifying an `<attr>` element, if it was previously defined you do not specify the format. If you wish to reuse an android attr, for example android:gravity, then you can do that in the name, as follows.

An example of a custom view `<declare-styleable>`:

```
<declare-styleable name="MyCustomView"> 
    <attr name="my_custom_attribute" />
    <attr name="android:gravity" />
</declare-styleable>
```

When defining you custom attributes in XML on you need to do a few things.  

First you must declare a namespace to find your attributes. You do this on the root layout element. Normal there is only `xmlns:android="http//schemas.android.com/apk/res/android"`. You must now also add `xmlns:app="http://schemas.android.com/apk/res-auto"`.

Example:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:whatever="http://schemas.android.com/apk/res-auto"
android:orientation="vertical"
android:layout_width="fill_parent"
android:layout_height="fill_parent">

<org.example.mypackage.MyCustomView
android:layout_width="fill_parent"
android:layout_height="wrap_content"
android:gravity="center"
whatever:my_custom_attribute="Hello, world!" />
</LinearLayout>
```

Finally, to access that custom attribute you normally do so in the constructor of you custom view as follows:

```
public MyCustomView(Context context, AttributeSet attrs, int defStyle) {
    super(context, attrs, defStyle);

    TypedArray a = context.obtainStyledAttributes(attrs, R.styleable.MyCustomView, defStyle, 0);

    String str = a.getString(R.styleable.MyCustomView_my_custom_attribute);

    //do something with str

    a.recycle();
}
```

Reference:[Defining custom attrs](http://stackoverflow.com/questions/3441396/defining-custom-attrs)  

###2. How to get Uri from raw file?
A: `Uri url = Uri.parse("android.resource://" + getPackageName() + "/" + R.raw.usa_for_africa_we_are_the_world);`  

Reference:[Android - How to get Uri from raw file?](http://stackoverflow.com/questions/16791439/android-how-to-get-uri-from-raw-file)  

###3. How to display all music on SD card?
A: We can use loader perform this work,

```
public Loader<Cursor> onCreateLoader(int id, Bundle args) {
    return new CursorLoader(getActivity(), MediaStore.Audio.Media.EXTERNAL_CONTENT_URI, new String[] { MediaStore.Audio.Media.DISPLAY_NAME }, null, null,
      "LOWER(" + MediaStore.Audio.Media.TITLE + ") ASC");    
}
```

Reference:[Display all music on SD card](http://stackoverflow.com/questions/8994625/display-all-music-on-sd-card) 

###4. Call require API level 21(current min is 10):android.content.Context#getDrawable.
A:`ContextCompat.getDrawable(getActivity(), R.drawable.name);`  

Reference:[Android getResources().getDrawable() deprecated API 22](http://stackoverflow.com/questions/29041027/android-getresources-getdrawable-deprecated-api-22/29041466#29041466)  

###5. Android Studio 工程卡在 Refreshing 'Project Name' Gradle project 或 gradle building project。
A: Android Studio 更新到 2.3 后，打开之前的项目卡在 Refreshing 'Project Name' Gradle project, 问题的原因是新版本的 Android Studio 使用的也是新版本的 Gradle,由于网络原因导致更新 Gradle 很慢，我们可以手动下载 Gradle 来解决这个问题。  

1. 根据项目中 gradle-wrapper.properties 文件的配置，手动下载合适版本的 gradle;

```
distributionUrl=https\://services.gradle.org/distributions/gradle-3.3-all.zip
```

2. 将下载好的 gradle 移动到合适的位置;

```
// Windows
C:\Users\用户名.gradle\wrapper\dists\gradle-x.xx-all\xxxxxxxxxxxx
// MacOS
$HOME/.gradle/wrapper/dists/gradle-3.3-all/55gk2rcmfc6p2dg9u9ohc3hw9/

```

3. 重启 Android Studio。

Reference:
[Android Studio解决refreshing gradle project](http://www.jianshu.com/p/3063173deed8)  
[Androidstudio配置本地Gradle](http://zzqhost.com/?post=82)  

###6. 怎么改变 EditText 输入框底部线条的颜色？
A: 在 style.xml 中覆盖 colorControlActivated, ColorControlHighlight 和 colorControlNormal

```
<style name="Theme.App.Base" parent="Theme.AppCompat.Light.DarkActionBar">
    <item name="colorControlNormal">#c5c5c5</item>
    <item name="colorControlActivated">@color/accent</item>
    <item name="colorControlHighlight">@color/accent</item>
</style>
```

Reference:[Changing EditText bottom line color with appcompat v7](https://stackoverflow.com/questions/26574328/changing-edittext-bottom-line-color-with-appcompat-v7)

###7.如何隐藏软键盘？
A:

```
// Check if no view has focus:
View view = this.getCurrentFocus();
if (view != null) {  
    InputMethodManager imm = (InputMethodManager)getSystemService(Context.INPUT_METHOD_SERVICE);
    imm.hideSoftInputFromWindow(view.getWindowToken(), 0);
}
```
Reference:[Close/hide the Android Soft Keyboard](https://stackoverflow.com/questions/1109022/close-hide-the-android-soft-keyboard)  

###8.ListView 中的 EditText 捕获焦点异常。
A:在 manifest 中配置下 Activity :

```
<activity android:name= ".yourActivity" android:windowSoftInputMode="adjustPan"/>
```

Reference:[Focusable EditText inside ListView](https://stackoverflow.com/questions/2679948/focusable-edittext-inside-listview)

###9.Android Studio 如何设置使用本地 gradle?
A:

1. 下载 gradle;
2. 解压 gradle 到合适的目录，例如 /Users/meiliang/.gradle/local_gradle_distribution/;
3. 在 Android Studio 中设置 gradle home: 

```
 Android Studio > Preference... > Build,Excution,Deployment > Gradle > User local gradle distribution > Gradle home

 Gradle home :/Users/meiliang/.gradle/local_gradle_distribution/gradle-3.3
```

###10.Could not find method leftShift() for arguments on task of type org.gradle.api.DefaultTask
A:<< was deprecated in 4.x and removed in 5.0.  

```
task createDB<<{
    description= "Creates the Database. Needs database name to be passed (like: -PdbName=someDBname)"
    sql = Sql.newInstance( 'jdbc:mysql:thin://localhost:3306/', mysqlUser, mysqlPassword, 'org.drizzle.jdbc.DrizzleDriver' )
    sql.execute( 'create database '+"`$dbName`" )
}

// Change to 
task createDB {
   description= "Creates the Database. Needs database name to be passed (like: -PdbName=someDBname)"
   doLast {    
    sql = Sql.newInstance( 'jdbc:mysql:thin://localhost:3306/', mysqlUser, mysqlPassword, 'org.drizzle.jdbc.DrizzleDriver' )
    sql.execute( 'create database '+"`$dbName`" )
   } 
}
```

Reference:[Could not find method leftShift() for arguments on task of type org.gradle.api.DefaultTask)[https://discuss.gradle.org/t/could-not-find-method-leftshift-for-arguments-on-task-of-type-org-gradle-api-defaulttask/30614)  

###11.How to unzip and view content of aar?
A:

```
$jar -xvf app-debug.aar
```

###12.How to update local aar module?
A:

1. Update yourLib.aar file in module, eg. copy new version aar to module dir
2. Sync Project with Gradle Files


