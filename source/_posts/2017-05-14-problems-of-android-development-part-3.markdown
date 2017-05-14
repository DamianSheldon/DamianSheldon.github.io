---
layout: post
title: "Android 开发问题汇总(三)"
date: 2017-05-14 16:56:19 +0800
comments: true
categories: [Archives, Android] 
keywords: Android, development 
discription: Problems of android development. 
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

Reference:[Add Rounded Corners to Views and Layouts Android](http://www.gadgetsaint.com/tips/rounded-corners-views-layouts-android/#.WRgR3FOGORt)  
