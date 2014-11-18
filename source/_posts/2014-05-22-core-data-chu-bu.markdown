---
layout: post
title: "Core Data 初步"
date: 2014-05-22 09:53:40 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Core Data, iOS, Database
description: Simple introduce of Core Data
---
1. Core Data是什么？
2. 为什么使用Core Data?
3. 如何使用Core Data?

##Core Data是什么？

>The Core Data framework provides generalized and automated solutions to common tasks associated with object life-cycle and object graph management, including persistence.

Core Data框架为关联对象生命周期和对象图管理的常见任务提供了通用的和自动的解决方案，包括持久化。

##为什么使用Core Data？
Core Data是一个对象图管理和持久化框架。它有以下优点：

* 允许你高效地从永久存储中获取模型对象和保存改变。
* 提供一个记录模型对象改变的架构。它能让你自动支持undo和redo，维护对象之间的关系。
* 允许你去维护模型对象不相交集合的编辑。不相交集合很有用，例如，让用户在一个可能会被丢弃的视图中编辑而不在另一个视图中显示的数据。
* 允许你在任何时候只在内存中保持模型对象的一个子集。这对保持你应用尽可能地使用少的内存很有用。
* 拥有数据存储版本和迁移的架构。该架构让你很容易把旧版本数据文件升级为现在的版本。

>There are a number of reasons why it may be appropriate for you to use Core Data. One of the simplest metrics is that, with Core Data, the amount of code you write to support the model layer of your application is typically 50% to 70% smaller as measured by lines of code. This is primarily due to the features listed above—the features Core Data provides are features you don’t have to implement yourself. Moreover they’re features you don’t have to test yourself, and in particular you don’t have to optimize yourself.


##如何使用Core Data?
Core Data 有相当多可用的组件,下图示意了这些组件之间的关系：
 
<div style="text-align:center" markdown="1">
	<img name="Core Data Stack" src="/images/CoreDataStack.jpg" width="610" height="418"> 

</div>

其实我们主要就是对数据进行CRUD操作，我们过一遍创建数据的过程，其他操作也就容易理解了。  

应用的需求会决定应该创建些什么模型类，通常我们会创建模型文件（后缀为xcdatamodeld，当然也可以纯代码），模型文件对应Managed Object Model（NSManagedObjectModel），它是实体的一个集合，实体和模型类一一对应。

应用启动以后会实例化一个Managed Object Context（NSManagedObjectContext），它相当于一个总入口，所有的数据操作都是通过它进行。Managed Object Context会持有一个Persistent Store Coordinator（NSPersistentStoreCoordinator）。

Persistent Store Coordinator帮我们将固化的文件内容映射成managed model,并且可以管理多个永久存储池（NSPersistentStore），这样managed object context看到的只是单个存储池。 

当所有的组件都捆绑到一起的时候，我们把它称作 Core Data 堆栈，这个堆栈有两个主要部分。一部分是关于对象图管理，这正是你需要很好掌握的那一部分，并且知道怎么使用。第二部分是关于持久化，比如，保存你模型对象的状态，然后再恢复模型对象的状态。

在两个部分之间，即堆栈中间，是持久化存储协调器（persistent store coordinator），也被称为中间审查者。它将对象图管理部分和持久化部分捆绑在一起，当它们两者中的任何一部分需要和另一部分交流时，这便需要持久化存储协调器来调节了。

对象图管理是你程序模型层的逻辑存在的地方。模型层的对象存在于一个 context 内。在大多数的设置中，存在一个 context ，并且所有的对象存在于那个 context 中。Core Data 支持多个 contexts，不过对于更高级的使用情况才用。注意每个 context 和其他 context 都是完全独立的，一会儿我们将会谈到。需要记住的是，对象和它们的 context 是相关联的。每个被管理的对象都知道自己属于哪个 context，并且每个 context 都知道自己管理着哪些对象。

堆栈的另一部分就是持久了，即 Core Data 从文件系统中读或写数据。每个持久化存储协调器（persistent store coordinator）都有一个属于自己的持久化存储（persistent store），并且这个 store 在文件系统中与 SQLite 数据库交互。为了支持更高级的设置，Core Data 可以将多个 stores 附属于同一个持久化存储协调器，并且除了存储 SQL 格式外，还有很多存储类型(binary,XML)可供选择。

##代码示例
纯理论的讲我们可能不容易理解，让我们结合代码来看看。
###创建和加载Managed Object Model
```
// 1)You usually create a model in Xcode, as described in Core Data Model Editor Help. 

[[NSManagedObjectModel alloc] initWithContentsOfURL:[[NSBundle mainBundle] URLForResource:@"NameOfCoreDataResource" withExtension:@"momd"]]

// Another method
managedObjectModel = [[NSManagedObjectModel mergedModelFromBundles:nil] retain];

// 2)You can also create a model entirely in code.
NSManagedObjectModel *mom = [[NSManagedObjectModel alloc] init];
NSEntityDescription *runEntity = [[NSEntityDescription alloc] init];
[runEntity setName:@"Run"];
[runEntity setManagedObjectClassName:@"Run"];
[mom setEntities:@[runEntity]];
 
NSMutableArray *runProperties = [NSMutableArray array];
 
NSAttributeDescription *dateAttribute = [[NSAttributeDescription alloc] init];
[runProperties addObject:dateAttribute];
[dateAttribute setName:@"date"];
[dateAttribute setAttributeType:NSDateAttributeType];
[dateAttribute setOptional:NO];
 
NSAttributeDescription *idAttribute= [[NSAttributeDescription alloc] init];
[runProperties addObject:idAttribute];
[idAttribute setName:@"processID"];
[idAttribute setAttributeType:NSInteger32AttributeType];
[idAttribute setOptional:NO];
[idAttribute setDefaultValue:@0];
 
NSPredicate *validationPredicate = [NSPredicate predicateWithFormat:@"SELF >= 0"];
NSString *validationWarning = @"Process ID < 0";
[idAttribute setValidationPredicates:@[validationPredicate]
    withValidationWarnings:@[validationWarning]];
 
[runEntity setProperties:runProperties];
 
NSDictionary *localizationDictionary = @{
    @"Property/processID/Entity/Run" : @"Process ID",
    @"Property/date/Entity/Run" : @"Date"
    @"ErrorString/Process ID < 0" : @"Process ID must not be less than 0" };
[mom setLocalizationDictionary:localizationDictionary];
```

###创建Managed Object Context
```
// Assume we have a persistent store coordinator
        managedObjectContext = [[NSManagedObjectContext alloc] init];
        [managedObjectContext setPersistentStoreCoordinator: coordinator];
```

###创建Persistent Store Coordinator
```
    NSURL *storeUrl = [NSURL fileURLWithPath: [[self applicationDocumentsDirectory] stringByAppendingPathComponent: @"Locations.sqlite"]];
	
	NSError *error;
    persistentStoreCoordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel: [self managedObjectModel]];
    if (![persistentStoreCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeUrl options:nil error:&error]) {
        // Handle the error.
    } 
```

###创建Persistent Store
我们不直接创建persistent object store。当我们给persistent store coordinator发送addPersistentStoreWithType:configuration:URL:options:error:, Core Data为我们创建合适的store.

##完整示例
上面是分解开来的示例，我们最后来看过完整的实例，我稍稍修改下苹果的官方示例[Locations](https://github.com/DamianSheldon/Locations).

##Reference
Core Data Programming Guide    
[Core Data 概述](http://objccn.io/issue-4-1/)   
[深入浅出 Cocoa 之 Core Data](http://blog.csdn.net/kesalin/article/details/6739319)    
[iOS开发系列--数据存取](http://www.cnblogs.com/kenshincui/p/4077833.html#CoreData)

