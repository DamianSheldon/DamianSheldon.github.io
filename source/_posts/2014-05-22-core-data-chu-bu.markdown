---
layout: post
title: "Core Data 初步"
date: 2014-05-22 09:53:40 +0800
comments: true
published: false
categories: [Archives, iOS Development]
keywords: Core Data, iOS, Database
description: Introduce Core Data
---
##Core Data是什么？
苹果官方对Core Data的介绍是这样的：

The Core Data framework provides generalized and automated solutions to common tasks associated with object life-cycle and object graph management, including persistence.

Core Data框架为关联对象生命周期和对象图管理的常见任务提供了通用的和自动的解决方案。

好吧，我们对它还是很陌生，让我们再深入一点。

你可以在应用中使用Core Data来管理模型（model-view-controller的model）。Core Data是一个对象图管理和持久化框架。它有这么一些优点：

* 允许你高效地从永久存储中获取模型对象和保存改变。
* 提供一个记录模型对象改变的架构。它能让你自动支持undo和redo，维护对象之间的关系。
* 允许你去维护模型对象不相交集合的编辑。不相交集合很有用，例如，让用户在一个可能会被丢弃的视图中编辑而不在另一个视图中显示的数据。
* 允许你在任何时候只在内存中保持模型对象的一个子集。这对保持你应用尽可能地使用少的内存很有用。
* 拥有数据存储版本和迁移的架构。该架构让你很容易把旧版本数据文件升级为现在的版本。


##如何使用Core Data?
Core Data有一系列类，它们共事的机制叫Core Data Stack。我们正是使用它来完成模型数据的持久化。简单的Core Data Stack可能是这样的：
<img name="Core Data Stack Simple" src="/images/stack-simple.png" width="550" height="293">  
复杂的Core Data Stack可能是这样的：  
<img name="Core Data Stack Complex" src="/images/stack-complex.png" width="624" height="652">  
让我们从上到下来学习涉及到的类。  
1)NSManagedObject：NSManagedObject及其子类的实例是模型对象，代表持久存储中的一条记录。  

2)NSManagedObjectContext：NSManagedObjectContext的实例叫做managed object context，它在Core Data应用中代表单个对象空间，或者便笺。 它的主要责任是管理managed objects集合。  

3)NSPersistentStoreCoordinator：NSPersistentStoreCoordinator帮我们将固化的文件内容映射成managed model,并且可以管理多个永久存储池，这样managed object context看到的只是单个存储池。  

4)NSPersistentStore：它是数据实际存储的仓库，Core Data支持三种类型的存储：binary, XML, and SQLite.我们也可以实现自已的存储类型。  

##代码示例
纯理论的讲我们可能不容易理解，让我们结合代码来看看。
###创建和加载Managed Object Model
``` objective-c
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
``` objective-c
// Assume we have a persistent store coordinator
        managedObjectContext = [[NSManagedObjectContext alloc] init];
        [managedObjectContext setPersistentStoreCoordinator: coordinator];
```

###创建Persistent Store Coordinator
``` objective-c
    NSURL *storeUrl = [NSURL fileURLWithPath: [[self applicationDocumentsDirectory] stringByAppendingPathComponent: @"Locations.sqlite"]];
	
	NSError *error;
    persistentStoreCoordinator = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel: [self managedObjectModel]];
    if (![persistentStoreCoordinator addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeUrl options:nil error:&error]) {
        // Handle the error.
    } 
```

###创建Persistent Store
我们不直接创建persistent object store。当我们给persistent store coordinator发送addPersistentStoreWithType:configuration:URL:options:error: Core Data为我们创建合适的store.

##完整示例
上面是分解开来的示例，我们最后来看过完整的实例，我们稍稍修改下苹果的官方示例[Locations](https://github.com/DamianSheldon/Locations).
