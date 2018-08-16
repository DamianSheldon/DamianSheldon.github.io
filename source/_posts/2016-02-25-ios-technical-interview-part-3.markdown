---
layout: post
title: "iOS 面试题汇总(三)"
date: 2016-02-25 18:43:44 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS, Technical, Interview, Category, Extension, KVO, KVC, shallow copy, deep copy
description: iOS Technical Interview questions about categories, extension, KVO, KVC, shallow copy, deep copy and so on.
---

###1.Difference between categories and extensions?
A:There are three difference beween categories and extensions:

1. The extension has no name;
2. The implementation of the extension must be in the main @implementation block of the file;
3. The extension can easy declare property that need backend instance variable.

Reference:[Difference between Category and Class Extension?](http://stackoverflow.com/questions/3499704/difference-between-category-and-class-extension)

###2.What is advantage of category?
A:

* You can add method to existing class even to that class whose source is not available to you. 
* You can extend functionality of a class without subclassing. 
* You can split implementation in multiple files. 

###3.What are KVO and KVC?
A:
> Key-value observing is a mechanism that enables an object to be notified directly when a property of another object changes.  

> Key-value coding is a mechanism for indirectly accessing an object’s attributes and relationships using string identifiers. 

Reference:Cocoa Core Competencies
<!-- more -->
###4.Difference between shallow copy and deep copy?
A:
The difference between shallow and deep copying is only relevant for compound objects (objects that contain other objects, like lists or class instances):

A shallow copy constructs a new compound object and then (to the extent possible) inserts references into it to the objects found in the original.
A deep copy constructs a new compound object and then, recursively, inserts copies into it of the objects found in the original.

Reference:[What exactly is the difference between shallow copy, deepcopy and normal assignment operation?](http://stackoverflow.com/questions/17246693/what-exactly-is-the-difference-between-shallow-copy-deepcopy-and-normal-assignm)

###5.When retain count increase?
A:

* When you create an object;
* When you send an object retain message;

Reference:[Conditions in which retainCount increases or decreases](http://stackoverflow.com/questions/4254346/conditions-in-which-retaincount-increases-or-decreases)

###6.If we don’t create any autorelease pool in our application then is there any autorelease pool already provided to us?
A: It depends on case, if we create app with UIKit or AppKit, there is a autorelease pool already provided to us; if we create command line app, we should create one by ourself.

###7.When you will create an autorelease pool in your application?
A:There are three occasions when you might use your own autorelease pool blocks:

* If you are writing a program that is not based on a UI framework, such as a command-line tool.
* If you write a loop that creates many temporary objects.
You may use an autorelease pool block inside the loop to dispose of those objects   before the next iteration. Using an autorelease pool block in the loop helps to  reduce the maximum memory footprint of the application.

* If you spawn a secondary thread.
You must create your own autorelease pool block as soon as the thread begins executing; otherwise, your application will leak objects.

###8.How many autorelease pool you can create in your application? Is there any limit?
A: You can create autorelease pool as much as possible, there is no limit.

###9.What is purpose of delegates?
A:Delegation is a simple and powerful pattern in which one object in a program acts on behalf of, or in coordination with, another object.

>委托是一种组合方法，它使组合具有与继承同样的复用能力。在委托方式下，有两个对象参与处理一个请求，接受请求的对象将操作委托给它的代理者（delegate），它类似于子类将请求交给它的父类处理。使用继承时，被继承的操作总能引用接受请求的对象。 --设计模式

###10.What is difference between NSNotification and protocol?
A:

* Protocol is To-one Relationships; NSNotification is To-many Relationships.
* Protocol support bidirectional data-transfer between two classes.
* Protocol doesn't need the third object to coordinate the communication.

###11.Polymorphism？
A: The ability of different objects to respond, each in its own way, to identical messages is called polymorphism.

###12.Singleton?
A:A singleton class returns the same instance no matter how many times an application requests it.

###13.Give us example of what are delegate methods and what are data source methods of UITableView?
A:

```objc
// Delegate methods
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath;
- (CGFloat)tableView:(UITableView *)tableView heightForRowAtIndexPath:(NSIndexPath *)indexPath;
- 
 // Data source methods
 - (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView;
 - (NSInteger)tableView:(UITalbleView *)tableView numberOfRowsInSections:(NSInteger)section;
 - (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath;

```

###14.What are mutable and immutable types in Objective C?
A:A mutable object can be mutated or changed. An immutable object cannot. 

Reference:[what is difference between mutable and immutable](http://stackoverflow.com/questions/7071096/what-is-difference-between-mutable-and-immutable)

###15.When to use NSMutableArray and when to use NSArray?
A:When collection will mutable you should use NSMutableArray, otherwise use NSArray.

###16.What is convenience constructor?
A:A convenience constructor is one that performs object allocation & initialization in one step & returns an autoreleased object to the caller.

Reference:[Objective-C (programming language): What is a convenience constructor](https://www.quora.com/Objective-C-programming-language/What-is-a-convenience-constructor)

###17.What is responder chain?
A:The responder chain is a series of linked responder objects. 

###18.What is push notification?
A:Apple Push Notification service (APNs for short) is the centerpiece of the remote notifications feature. It is a robust and highly efficient service for propagating information to iOS and OS X devices. Each device establishes an accredited and encrypted IP connection with the service and receives notifications over this persistent connection. If a notification for an app arrives when that app is not running, the device alerts the user that the app has data waiting for it.


