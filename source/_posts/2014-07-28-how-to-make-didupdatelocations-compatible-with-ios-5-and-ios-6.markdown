---
layout: post
title: (翻译)如何让didUpdateLocation兼容iOS 5和iOS 6
date: 2014-07-28 16:05:38 +0800
comments: true
categories: [iOS Development, Archives]
keywords: Core Location, iOS, compatible
description: 如何让didUpdateLocation兼容iOS 5和iOS 6
---

`- (void)locationManager:(CLLocationManager *)manager didUpdateToLocation:(CLLocation *)newLocation fromLocation:(CLLocation *)oldLocation`是CLLocationManagerDelegate protocol中的一个常用方法，它让你的应用接收更新位置信息，当检测到任何位置变化。新的位置详情存储在newLocation中，它是一个CLLocation.  
当iOS6启动，上述方法被废弃了，建议使用新版本方法`- (void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations`或简称locationManager:didUpdateLocations:.  

这个快速教程的的创建目的是为了解释如何处理废弃方法，关于它什么是应该完成的以及你在哪里可以找出关于改变的更多细节。如果你想了解如何使用这个新的locationManager:didUpdateLocations: 方法，看下[didUpdateLocations tutorial](http://www.devfright.com/didupdatelocations-ios-example/)，它解释了如何使用方法提供的NSArray.  

###当方法被废弃了
当iOS升级了（这很经常），Apple找到新的或更有效方法。当这发生了，方法可以被标记为废弃并在如何使用的地方给出一个提示。具体到CLLocationManagerDelegate，你可以看到文档中推荐了一个不同的方法。虽然你仍然可以使用废弃的方法，即使是在已经废弃的iOS版本中，Apple在未来某个时间点也许会删除掉这个方法当iOS升级了。在那个时间点，你可能需要修改你的代码，提交到Apple Store通过审核流程。  

<!-- more -->

###如何处理废弃的方法
已经被废弃的方法在新的iOS版本上仍然可以工作。Apple趋向于让它们在未来的几个新版本中保持可用，然后再将它们从类或协议中删除。与其让你的代码在最后关头更新或重新提交到苹果商店。你可以在改变之前做好准备。  

今天的例子，我们会看下 locationManager:didUpdateToLocation:fromLocation:方法以及如何让旧的废弃方法和新的方法在同一份代码中共同工作。  
``` objective-c

-(void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations {
    CLLocation *newLocation = [locations lastObject];
    CLLocation *oldLocation = [locations objectAtIndex:locations.count-1];
    NSLog(@"didUpdateToLocation %@ from %@", newLocation, oldLocation);
    MKCoordinateRegion userLocation = MKCoordinateRegionMakeWithDistance(newLocation.coordinate, 1500.0, 1500.0);
    [regionsMapView setRegion:userLocation animated:YES];
}
```
为了兼容iOS 5,我们可以加入旧的方法`- (void)locationManager:(CLLocationManager *)manager didUpdateToLocation:(CLLocation *)newLocation fromLocation:(CLLocation *)oldLocation`，并在旧的方法中调用新的方法，代码看起来会像这样：
``` objective-c
-(void)locationManager:(CLLocationManager *)manager didUpdateLocations:(NSArray *)locations {
    CLLocation *newLocation = [locations lastObject];
    CLLocation *oldLocation;
    if (locations.count >= 2) {
        oldLocation = [locations objectAtIndex:locations.count-1];
    } else {
        oldLocation = nil;
    }
    NSLog(@"didUpdateToLocation %@ from %@", newLocation, oldLocation);
    MKCoordinateRegion userLocation = MKCoordinateRegionMakeWithDistance(newLocation.coordinate, 1500.0, 1500.0);
    [regionsMapView setRegion:userLocation animated:YES];
}

- (void)locationManager:(CLLocationManager *)manager didUpdateToLocation:(CLLocation *)newLocation fromLocation:(CLLocation *)oldLocation {
    [self locationManager:locationManager didUpdateLocations:[[NSArray alloc] initWithObjects:newLocation, nil]];
}
```

如果是iOS5，新的方法不会被调用，它就像自定义的方法，iOS5并不知道locationManager:didUpdateLocations:事实上是什么。旧的方法简单的alloc/init新的名为currentLocation的NSArray并且使用 initWithObjects:newLocation获取一个newLocation,然后NSArray作为信息被传递到locationManager新的方法。  

总之，如果设备是iOS5,旧的方法locationManager:didUpdateToLocation:fromLocation: 告诉代理新的位置可用，它把CLLocation加到NSArray中传递给新的方法，但设备的操作系统并不知道新方法。  

如果使用的是iOS6,我们从CLLocationManagerDelegate的头文件中了解到如果代码中既有旧方法又有新方法，那么iOS会调用新方法通知代理位置更新了。

虽然这个快速教程提供了一种处理废弃方法的办法，仍然会有其他很多不同的办法。另外，多查看文档和关文件，因为Apple添加了很多信息，它们很有可能会为你手头的任务提供解决办法。

###原文  
o [How to Make didUpdateLocations Compatible with iOS 5 and iOS 6](http://www.devfright.com/how-to-make-didupdatelocations-compatible-with-ios-5-and-ios-6/)


