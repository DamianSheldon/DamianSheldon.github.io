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

>Core Data is a framework that you use to manage the model layer objects in your application. It provides generalized and automated solutions to common tasks associated with object life-cycle and object graph management, including persistence.

Core Data 是一个你在应用中用来管理模型层对象的框架。 它为关联对象生命周期和对象图管理的常见任务提供了通用的和自动的解决方案，包括持久化。

##为什么使用Core Data？

>There are a number of reasons why it may be appropriate for you to use Core Data. One of the simplest metrics is that, with Core Data, the amount of code you write to support the model layer of your application is typically 50% to 70% smaller as measured by lines of code. This is primarily due to the features listed above—the features Core Data provides are features you don’t have to implement yourself. Moreover they’re features you don’t have to test yourself, and in particular you don’t have to optimize yourself.

Core Data 的优点：

* 允许你高效地从永久存储中获取模型对象和保存改变。
* 提供一个记录模型对象改变的架构。它能让你自动支持undo和redo，维护对象之间的关系。
* 允许你去维护模型对象不相交集合的编辑。不相交集合很有用，例如，让用户在一个可能会被丢弃的视图中编辑而不在另一个视图中显示的数据。
* 允许你在任何时候只在内存中保持模型对象的一个子集。这对保持你应用尽可能地使用少的内存很有用。
* 拥有数据存储版本和迁移的架构。该架构让你很容易把旧版本数据文件升级为现在的版本。

<!-- more -->



##如何使用 Core Data?

使用 Core Data 之前需要初始化 Core Data Stack, 它会为我们处理所有和底层数据的交互，而我们只需要关注自己的业务逻辑。

>The Core Data stack is a collection of framework objects that are accessed as part of the initialization of Core Data and that mediate between the objects in your application and external data stores.

Core Data Stack 示意图：
 
<div style="text-align:center" markdown="1">
	<img name="Core Data Stack" src="/images/CoreDataStack.jpg" width="410" height="281"> 

</div>

* An external persistent store that contains saved records.
* A persistent object store that maps between records in the store and objects in your application.
* A persistent store coordinator that aggregates all the stores.
* A managed object model that describes the entities in the stores.
* A managed object context that provides a scratch pad for managed objects.
* A managed object is a model object (in the model-view-controller sense) that represents a record from a persistent store. 

* Store File: 顾名思义即存储文件，它可以是 SQLite, XML 或 binary 文件；
* Persistent Object Store: 它负责把从 Store File 读取数据映射成应用能理解的记录，把应用产生的记录根据配置的存储类型写入到 Store File 中；
* Persistent Store Coordinator: 它负责协调有多个 Persistent Object Store 的情况；
* Managed Object Model: 它是实体描述的集合，实体类似于数据库中的表；
* Managed Object Context: 它为 managed objects 提供暂存的空间，也是应用操作 Core Data 的接口；
* Managed Object: 它是代表某个实体的模型对象。

我是这样理解它们的：Store File 是具体存放二进制数据的地方，二进制数据也有组织方式，所以会引入 SQLite, XML 或 binary，直接操作二进制文件肯定不方便，那自然想到用一个对象来做这个事情，于是引入 Persistent Object Store，这样底层二进制的操作对上层来说就透明了。实体在这里就是一个能被双方理解的数据结构，本来它应该被直接被 Persistent Object Store 持有，有了支持多个 Store 的 Persistent Store Coordinator 后，让 coordinator 来持有它，显得更合理。Managed Object Model 就是所有实体描述的集合。Managed Object 是代表某个实体的模型对象，实体有个类名来关联它。

在具体的使用时，我们可以通过 IB 或者纯代码方式。

### Interface Builder 方式
* 用 Interface Builder 方式创建 Managed Object Model

* 初始化 Core Data Stack

```
- (void)initializeCoreData
{
    NSURL *modelURL = [[NSBundle mainBundle] URLForResource:@"Workspace" withExtension:@"momd"];
    NSManagedObjectModel *mom = [[NSManagedObjectModel alloc] initWithContentsOfURL:modelURL];
    NSAssert(mom != nil, @"Error initializing Managed Object Model");
    
    NSPersistentStoreCoordinator *psc = [[NSPersistentStoreCoordinator alloc] initWithManagedObjectModel:mom];
    NSManagedObjectContext *moc = [[NSManagedObjectContext alloc] initWithConcurrencyType:NSMainQueueConcurrencyType];
    [moc setPersistentStoreCoordinator:psc];
    [self setManagedObjectContext:moc];
    
    NSFileManager *fileManager = [NSFileManager defaultManager];
    NSURL *documentsURL = [[fileManager URLsForDirectory:NSDocumentDirectory inDomains:NSUserDomainMask] lastObject];
    NSURL *storeURL = [documentsURL URLByAppendingPathComponent:@"Workspace.sqlite"];
    
    dispatch_async(dispatch_get_global_queue( DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^(void) {
        NSError *error = nil;
        NSPersistentStoreCoordinator *psc = [[self managedObjectContext] persistentStoreCoordinator];
        NSPersistentStore *store = [psc addPersistentStoreWithType:NSSQLiteStoreType configuration:nil URL:storeURL options:nil error:&error];
        NSAssert(store != nil, @"Error initializing PSC: %@\n%@", [error localizedDescription], [error userInfo]);
    });
}
```

* 对数据进行基本的 CRUD 操作

```
// 1. Create
- (BOOL)insertDepartment:(NSDictionary *)department error:(NSError **)error
{
    NSManagedObject *departmentMO = [NSEntityDescription insertNewObjectForEntityForName:@"Department" inManagedObjectContext:self.managedObjectContext];
    [departmentMO setValue:department[@"name"] forKey:@"name"];
    NSArray *employees = department[@"employees"];
    if (employees) {
        [departmentMO setValue:employees forKey:@"employees"];
    }
    
    return [self.managedObjectContext save:error];
}

// 2. Read
- (NSArray *)departmentsMayResultWithError:(NSError **)error
{
    /*
     Fetch existing departments.
     Create a fetch request for the Event entity; add a sort descriptor; then execute the fetch.
     */
    NSFetchRequest *request = [[NSFetchRequest alloc] initWithEntityName:@"Department"];
    
    // Order the department by alpha.
    NSSortDescriptor *sortDescriptor = [[NSSortDescriptor alloc] initWithKey:@"name" ascending:YES];
    NSArray *sortDescriptors = @[sortDescriptor];
    [request setSortDescriptors:sortDescriptors];
    
    // Execute the fetch.
    NSArray *fetchResults = [self.managedObjectContext executeFetchRequest:request error:error];
    
    return fetchResults;
}

// 3. Update 
// Update department's property
department.name = ...
department.employees = ...

- (BOOL)updateDepartment:(NSManagedObject *)department error:(NSError **)error
{
    return [self.managedObjectContext save:error];
}

// 4. Delete
- (BOOL)deleteDepartment:(NSManagedObject *)department error:(NSError **)error
{
    [self.managedObjectContext deleteObject:department];
    
    return [self.managedObjectContext save:error];
}
```


### 代码示例
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
上面是分解开来的示例，我们最后来看几个完整的实例:

* [Core Data Demo](https://github.com/DamianSheldon/CoreDataDemo)
* [Locations](https://github.com/DamianSheldon/Locations).

##Reference
Core Data Programming Guide    
[Core Data 概述](http://objccn.io/issue-4-1/)   
[深入浅出 Cocoa 之 Core Data](http://blog.csdn.net/kesalin/article/details/6739319)    
[iOS开发系列--数据存取](http://www.cnblogs.com/kenshincui/p/4077833.html#CoreData)

