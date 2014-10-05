---
layout: post
title: "Core Animation 笔记"
date: 2014-09-03 09:59:58 +0800
comments: true
categories: [Archive, iOS Development]
keywords: Core Animation, iOS Development
description: Learning Core Animation Notes
---
##Core Animation介绍
Core Animation is a graphics rendering and animation infrastructure available on both iOS and OS X that you use to animate the views and other visual elements of your app.

##什么时候使用Core Animation? 
In places where you want to perform more sophisticated animations, or animations not supported by the UIView class, you can use Core Animation and the view’s underlying layer to create the animation. Because view and layer objects are intricately linked together, changes to a view’s layer affect the view itself.

##如何使用Core Animation?
  1)Enabling Core Animation Support in Your App;  
  In iOS apps, Core Animation is always enabled and every view is backed by a layer.iOS apps must link against this framework only if they use Core Animation interfaces explicitly.

  2)Selecting a appropriate Layer Object Associated with a View;  
  Reference:Core Animation Programming Guide -- Different Layer Classes Provide Specialized Behaviors

  3)Add animation to layer.  
  CABasicAnimation provides basic, single-keyframe animation capabilities for a layer property.  
  The CAKeyframeAnimation class provides keyframe animation capabilities for a layer object.

