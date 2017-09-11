---
layout: post
title: "Core Data 使用笔记"
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


### 代码方式

* 初始化 Core Data Stack

```
// Prepare Managed Object Model

- (NSManagedObjectModel *)employeesMOM
{
    static NSManagedObjectModel *sEmployeesMOM;
    
    if (sEmployeesMOM) {
        return sEmployeesMOM;
    }
    
    sEmployeesMOM = [NSManagedObjectModel new];
    
    // Create the entity
    NSEntityDescription *personEntity = [self entityOfPerson];
    
    NSEntityDescription *employeeEntity = [self entityOfEmployee];
    
    NSEntityDescription *departmentEntity = [self entityOfDepartment];
    
    // Construct entity inherit
    personEntity.subentities = @[employeeEntity];
    
    
    // Construct relationship
    NSRelationshipDescription *department = employeeEntity.relationshipsByName[@"department"];
    
    NSRelationshipDescription *employees = departmentEntity.relationshipsByName[@"employees"];
    
    department.destinationEntity = departmentEntity;
    department.inverseRelationship = employees;
    
    employees.destinationEntity = employeeEntity;
    employees.inverseRelationship = department;
    
    sEmployeesMOM.entities = @[personEntity, employeeEntity, departmentEntity];
    
    return sEmployeesMOM;
}

- (NSEntityDescription *)entityOfPerson
{
    NSEntityDescription *personEntity = [NSEntityDescription new];
    personEntity.name = @"Person";
    personEntity.managedObjectClassName = @"NSManagedObject";
    personEntity.abstract = YES;
    
    NSAttributeDescription *attributeOfName = [NSAttributeDescription new];
    attributeOfName.name = @"name";
    attributeOfName.attributeType = NSStringAttributeType;
    
    NSAttributeDescription *attributeOfDateOfBirth = [NSAttributeDescription new];
    attributeOfDateOfBirth.name = @"dateOfBirth";
    attributeOfDateOfBirth.attributeType = NSDateAttributeType;
    
    personEntity.properties = @[attributeOfName, attributeOfDateOfBirth];
    
    return personEntity;
}

- (NSEntityDescription *)entityOfEmployee
{
    NSEntityDescription *employeeEntity = [NSEntityDescription new];
    employeeEntity.name = @"Employee";
    employeeEntity.managedObjectClassName = @"Employee";
    
    NSAttributeDescription *attributeOfStartDate = [NSAttributeDescription new];
    attributeOfStartDate.name = @"startDate";
    attributeOfStartDate.attributeType = NSDateAttributeType;
    
    NSRelationshipDescription *department = [NSRelationshipDescription new];
    department.name = @"department";
    department.deleteRule = NSNullifyDeleteRule;
    department.minCount = 0;
    department.maxCount = 1;// max = 1 for to-one relationship
    
    employeeEntity.properties = @[attributeOfStartDate, department];
    
    return employeeEntity;
}

- (NSEntityDescription *)entityOfDepartment
{
    NSEntityDescription *departmentEntity = [NSEntityDescription new];
    departmentEntity.name = @"Department";
    departmentEntity.managedObjectClassName = @"NSManagedObject";
    
    NSAttributeDescription *attributOfName = [NSAttributeDescription new];
    attributOfName.name = @"name";
    attributOfName.attributeType = NSStringAttributeType;
    
    NSRelationshipDescription *employees = [NSRelationshipDescription new];
    employees.name = @"employees";
    employees.deleteRule = NSNullifyDeleteRule;
    employees.minCount = 0;
    employees.maxCount = 0;// max = 0 for to-many relationship
    
    departmentEntity.properties = @[attributOfName, employees];
    
    return departmentEntity;
}

// Initialize Core Data
- (void)initializeCoreData
{
    NSManagedObjectModel *mom = [self employeesMOM];

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

* 数据操作和 IB 方式是一样的

* personEntity.managedObjectClassName = "NSManagedObject"

>In some respects, an NSManagedObject acts like a dictionary—it is a generic container object that efficiently provides storage for the properties defined by its associated NSEntityDescription object. NSManagedObject provides support for a range of common types for attribute values, including string, date, and number (see NSAttributeDescription for full details). There is therefore commonly no need to define instance variables in subclasses. Sometimes, however, you want to use types that
>are not supported directly, such as colors and C structures. For example, in a graphics application you might want to define a Rectangle entity that has attributes color and bounds that are an instance of NSColor and an NSRect struct respectively. For some types you can use a transformable attribute, for others this may require you to create a subclass of NSManagedObject—see Non-Standard Persistent Attributes.

##完整示例
上面是分解开来的示例，我们最后来看几个完整的实例:

* [Core Data Demo](https://github.com/DamianSheldon/CoreDataDemo)
* [Locations](https://github.com/DamianSheldon/Locations).

##Reference
Core Data Programming Guide    
[Core Data 概述](http://objccn.io/issue-4-1/)   
[深入浅出 Cocoa 之 Core Data](http://blog.csdn.net/kesalin/article/details/6739319)    
[iOS开发系列--数据存取](http://www.cnblogs.com/kenshincui/p/4077833.html#CoreData)

