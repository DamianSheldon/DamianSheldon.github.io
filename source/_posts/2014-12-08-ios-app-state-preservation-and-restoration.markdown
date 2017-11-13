---
layout: post
title: "iOS App State Preservation and Restoration"
date: 2014-12-08 11:55:48 +0800
comments: true
published: false
categories: [Archives, iOS Development]
keywords: State Preservation Resoration
discription: Introduce iOS App State Preservation and Restoration
---
1. 什么是State Preservation and Restoration？
2. 为什么要用State Preservation and Restoration？
3. 如何使用State Preservation and Restoration？

###什么是State Preservation and Restoration？

State Preservation and Restoration组成UIKit的state preservation system，它提供简单而灵活的架构来保存和恢复应用的视图控制器和视图。


###为什么要用State Preservation and Restoration？

>Even if your app supports background execution, it cannot run forever. At some point, the system might need to terminate your app to free up memory for the current foreground app. However, the user should never have to care if an app is already running or was terminated. From the user’s perspective, quitting an app should just seem like a temporary interruption. When the user returns to an app, that app should always return the user to the last point of use, so that the user can continue with whatever task was in progress. This behavior provides a better experience for the user and with the state restoration support built in to UIKit is relatively easy to achieve.

即使你的应用支持后台运行，它也不可能一直运行。某个时刻，系统也许需要终止你的应用为当前应用释放内存。然而，用户应该永远不用关心应用是已经运行或终止。从用户的角度来看，退出应用应该就像暂时的中断。当用户回到应用，它应该回到上次使用的状态，以便用户能继续任何正在进行的任务。这种行为为用户提供了更好的体验，UIKit内置了这种状态恢复支持很容易就能实现。

<!-- more -->

###如何使用State Preservation and Restoration？

State Restoreation成功的前提条件如下：   
1. **状态保留必须成功。**也就是说应用终止前必须进入后台运行状态。   
2. **应用不可以强制退出。**   
3. **从最近一次状态保留开始，应用必须没有启动失败过。**   

Checklist for Implementing State Preservation and Restoration

1. (Required) Implement the `application:shouldSaveApplicationState:` and `application:shouldRestoreApplicationState: `methods in your app delegate; 
2. (Required) Assign restoration identifiers to each view controller you want to preserve by assigning a non empty string to their `restorationIdentifier` property;If you want to save the state of specific views too, assign non empty strings to their `restorationIdentifier` properties;
3. (Required) Show your app’s window from the `application:willFinishLaunchingWithOptions:` method of your app delegate. The state restoration machinery needs the window so that it can restore scroll positions and other relevant bits of your app’s interface.
4. Assign restoration classes to the appropriate view controllers. (If you do not do this, your app delegate is asked to provide the corresponding view controller at restore time.)
5. (Recommended) Encode and decode the state of your views and view controllers using the `encodeRestorableStateWithCoder:` and `decodeRestorableStateWithCoder:` methods of those objects;
6. Encode and decode any version information or additional state information for your app using the `application:willEncodeRestorableStateWithCoder:` and `application:didDecodeRestorableStateWithCoder:` methods of your app delegate;
7. Objects that act as data sources for table views and collection views should implement the **UIDataSourceModelAssociation** protocol. Although not required, this protocol helps preserve the selected and visible items in those types of views.

###注意事项

<div style="text-align: center" markdown="1">

	<img name="state_vc_caveats_2x" src="/images/state_vc_caveats_2x.png" width="594" height="862">

</div>
 
上图有颜色的视图控制器的状态是不会被保存和恢复的。

###Reference

App Programming Guide for iOS
