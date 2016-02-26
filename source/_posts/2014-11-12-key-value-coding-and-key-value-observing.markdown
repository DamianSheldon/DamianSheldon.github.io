---
layout: post
title: "Key-Value Coding &amp; Key-Value Observing"
date: 2014-11-12 16:06:13 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Key-Value Coding, Key-Value Observing
discription: Simple summary of Key-Value Coding And Key-Value Observing
---
##Key-Value Coding
1. Key-Value Coding是什么？
2. 为什么要用Key-Value Coding？
3. 如何使用Key-Value Coding?


###Key-Value Coding是什么？
> Key-value coding is a mechanism for accessing an object’s properties indirectly, using strings to identify properties, rather than through invocation of an accessor method or accessing them directly through instance variables.

Key-value coding是一种间接访问对象属性和机制，使用字符串去区别属性，而不是通过调用存取方法或者通过实例变量直接访问它们。


###为什么要用Key-Value Coding？
1. Scripting（OS X）;
2. 简化代码和提高灵活性；

###如何使用Key-Value Coding?

Key-value coding支持对象属性，也可以是标量类型和结构体。在开始使用之前，我们先熟悉些下key-value coding术语，后文我们用术语来阐述，方便交流。

除了重载现有的术语，key-value coding定义了些专属的术语。

Key-value coding可以被用来访问三种不同的类型的对象值：**attributes**, **to-one relationships**, 和**to-many relationships**。术语**property**指三种类型值的任意一种。

attribute是简单值的property，像标量，字符串，或者布尔值。NSNumber和其他的不可变类型如NSColor也都被认为是attributes。

to-one relationship是拥有自己properties的对象。这些内部的properties可以改变而对象却不变。例如，NSView实例的superview就是to-one relationship。

to-many relationship包括一个相关对象的集合。NSArray或NSSet经常被用来持有这样一个集合。但是，key-value coding允许你使用自定义的类作为集合并且通过实现在to-many Properties中讨论的key-value coding存取方法仍然可以可以像它们是NSArray或NSSet那样访问它们。

既然key-value coding是一种间接访问对象属性的机制，访问就包括存取。

<!-- more -->

####使用key-value coding读取attribute值
NSKeyValueCoding中定义的的attribute读取方法有：

```
- (id)valueForKey:(NSString *)key;

- (NSMutableArray *)mutableArrayValueForKey:(NSString *)key;

- (id)valueForKeyPath:(NSString *)keyPath;

- (NSMutableArray *)mutableArrayValueForKeyPath:(NSString *)keyPath;

- (NSDictionary *)dictionaryWithValuesForKeys:(NSArray *)keys;

- (id)valueForUndefinedKey:(NSString *)key;

```

####使用key-value coding存储attribute值

```
- (void)setValue:(id)value forKey:(NSString *)key;

- (NSMutableSet *)mutableSetValueForKey:(NSString *)key;

- (void)setValue:(id)value forKeyPath:(NSString *)keyPath;

- (NSMutableSet *)mutableSetValueForKeyPath:(NSString *)keyPath;

- (void)setValue:(id)value forUndefinedKey:(NSString *)key;

- (void)setNilValueForKey:(NSString *)key;

- (void)setValuesForKeysWithDictionary:(NSDictionary *)keyedValues;

```

你需要考虑一个额外的问题就是当你尝试设置非对象的property为nil。这种情况下，接收者给它自己发送一个setNilValueForKey:消息。它默认的实现是抛出NSInvalidArgumentException异常。你的应用可以覆盖这个方法替换默认值或标记值，然后你用新值调用setValue:forKey:。

现在，我们知道怎么用key-value coding去间接访问对象的property了。但是，如果想让我们自定义的类的property也支持key-value coding的话，我们应该怎么做呢？

####如何让自定义类的属性支持key-value coding?

>Key-value coding attempts to use accessor methods to get and set values, before resorting to directly accessing the instance variable.

在依靠直接访问实例变量之前，key-value coding尝试使用存取方法去访问和设置值。也就是说我们最好实现相应的存取方法，至少也要存在合适的实例变量。

前面提到key-value coding支持三个类型的attribute,针对三种类型的attribute,key-value coding分别是怎么来搜索存取方法和实例变量的呢？

1. 简单attributes存取方法搜索模式；
2. 顺序集合存取方法的搜索模式；
3. 唯一顺序集合存取方法的搜索模式；
4. 无序集合存取方法的搜索模式。

搜索的细节可以查阅Key-Value Coding Programming Guide。


##Key-Value Observing

1. Key-Value Observing是什么？
2. 为什么要用Key-Value Observing？
3. 如何使用Key-Value Observing?

###Key-Value Observing是什么？

>Key-value observing is a mechanism that allows objects to be notified of changes to specified properties of other objects.

Key-value observing是一种允许一对对象的特定属性改变时另一个对象被通知的机制。


###为什么要用Key-Value Observing？

KVO的主要好处是你不需要实现一套属性每次改变发送通知的机制。

###如何使用Key-Value Observing?

1. Registering as an Observer;
2. Receiving Notification of a Change;
3. Removing an Object as an Observer.

同样，自定义的类如何才能让其他的开发者能使用KVO呢？我们要做的是符合KVO标准。

####KVO Compliance

* The class must be **key-value coding compliant** for the property, as specified in **Ensuring KVC Compliance** in **Key-Value Coding Programming Guide**.
* The class emits KVO change notifications for the property.
* Dependent keys are registered appropriately (see **Registering Dependent Keys**).

---
There are two techniques for ensuring the change notifications are emitted. Automatic support is provided by NSObject and is by default available for all properties of a class that are key-value coding compliant. Typically, if you follow standard Cocoa coding and naming conventions, you can use automatic change notifications—you don’t have to write any additional code.

Manual change notification provides additional control over when notifications are emitted, and requires additional coding. You can control automatic notifications for properties of your subclass by implementing the class method automaticallyNotifiesObserversForKey:.

---

####Observing Changes to a mutable array using KVO

* Implement a method named `-<key>` that returns an array
* Implement method `-insertObject:in<Key>AtIndex:`
* Implement method `-removeObjectFrom<Key>AtIndex:`

> Although your application can implement accessor methods for to-many relationship properties using the -<key> and -set<Key>: accessor forms, you should typically only use those to create the collection object. For manipulating the contents of the collection it is best practice to implement the additional accessor methods referred to as the collection accessor methods. You then use the collection accessor methods, or a mutable collection proxy returned by mutableArrayValueForKey: or mutableSetValueForKey:.

```
NSMutableArray *array = [self.KVOArray mutableArrayValueForKey:@"KVOArray"];
        
[array addObject:anObject];
```

[KVO Array Demo](https://github.com/DamianSheldon/KeyValueObservingDemo.git)  

Extension:[如何自己动手实现 KVO](http://tech.glowing.com/cn/implement-kvo/)

####One more thing

Key-Value Observing Implementation Details

Automatic key-value observing is implemented using a technique called **isa-swizzling**.

The **isa** pointer, as the name suggests, points to the object's class which maintains a dispatch table. This dispatch table essentially contains pointers to the methods the class implements, among other data.

When an observer is registered for an attribute of an object the isa pointer of the observed object is modified, pointing to an intermediate class rather than at the true class. As a result the value of the isa pointer does not necessarily reflect the actual class of the instance.

You should never rely on the **isa** pointer to determine class membership. Instead, you should use the **class** method to determine the class of an object instance.

So how does **class** method determine the class of an object?


#Reference

Key-Value Coding Programming Guide   
Key-Value Observing Programming Guide


