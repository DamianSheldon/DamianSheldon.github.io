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

我们先熟悉些下key-value coding术语，后文我们用术语来阐述，方便交流。

除了重载现有的术语，key-value coding定义了些专属的术语。

Key-value coding可以被用来访问三种不同的类型的对象值：**attributes**, **to-one relationships**, 和**to-many relationships**。术语**property**指三种类型值的任意一种。

attribute 是简单值的 property，像标量，字符串，或者布尔值。NSNumber和其他的不可变类型如NSColor也都被认为是attributes。

to-one relationship 是拥有自己properties的对象。这些内部的properties可以改变而对象却不变。例如，NSView实例的superview就是to-one relationship。

to-many relationship 包括一个相关对象的集合。NSArray或NSSet经常被用来持有这样一个集合。但是，key-value coding允许你使用自定义的类作为集合并且通过实现在to-many Properties中讨论的key-value coding存取方法仍然可以可以像它们是NSArray或NSSet那样访问它们。

要使用 Key-Value Coding，对象要符合 Key-Value Coding Compliant，符合 Key-Value Coding Compliant 则有两点要求：一是对象遵守 NSKeyValueCoding 协议；二是要实现规定的方法。

因为 NSObject 已经遵守 NSKeyValueCoding 协议，所以类继承 NSObject 这个要求就默认满足了。于是重点就是实现规定的方法。那么规定的方法是哪些呢？

对于 Attribute 和 To-One Relationship Compliance  

* 实现 `<key>` 或者 `is<Key>`, 或者创建 `<key>` 或者 `_<key>` 实例变量。
* 如果属性可变，实现 `set<Key>` 方法。
* 如果属性是标量，覆盖 `setNilValueForKey: ` 方法去优雅处理 nil 。

对于 to-many relationship 的属性，实现上述方法后，我们就可以对集合对象本身使用 KVC 了。但是如果我们还实现额外规定的集合存取方法，我们可以：

* 用 NSArray 或 NSSet 之外的类为 to-many relationships 建模。
* 改变 to-many relationships 时性能更好。
* 提供 Key-Value observing compliant 访问你对象的集合属性的内容。

额外规定的集合存取方法如下：

* Accessing Indexed Collections

	* Indexed Collection Getters

		* `countOf<Key>`
		* `objectIn<Key>AtIndex:` or `<key>AtIndexes:`
		* (可选)`get<Key>:range:`

	* Indexed Collection Mutators
	
		* `insertObject:in<Key>AtIndex:` or `insert<Key>:atIndexes:`
		* `removeObjectFrom<Key>AtIndex:` or `remove<Key>AtIndexes:`
		* (可选)`replaceObjectIn<Key>AtIndex:withObject:` or 			`replace<Key>AtIndexes:with<Key>:`

* Accessing Unordered Collections

	* Unordered Collection Getters

		* `countOf<Key>`
		* `enumeratorOf<Key>`
		* `memberOf<Key>:`

	* Unordered Collection Mutators
	
		* `add<Key>Object:` or `add<Key>:`
		* `remove<Key>Object:` or `remove<Key>:`
		* (可选)`intersect<Key>:`

<!-- more -->


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

对 KVO-compliant 属性我们需要执行下列操作来使能对象接收到 key-value observing 通知：

* 对观察对象使用方法 `addObserver:forKeyPath:options:context:` 注册观察者。
* 在观察者内部实现 `observeValueForKeyPath:ofObject:change:context:` 来接受改变通知消息。
* 不再需要接收消息时使用方法 `removeObserver:forKeyPath:` 注销观察者。至少在观察者从内存中释放之前调用该方法。

那么属性什么时候认为是 KVO-compliant?

类必须确保以下内容来使属性 KVO-compliant：

* 类必须对属性是 key-value coding compliant。
* 类为属性发射 KVO 改变通知。
* 合适的注册了依赖 key 。

有两种技术发射改变通知。 NSObject 提供自动支持而且默认对类 key-value coding compliant 的属性是可用的。通常如果你遵从 Cocoa 编码和命名惯例，你不用编写任务额外代码就可以使用自动的改变通知。

手动改变通知可以在发射通知时提供更细腻的控制，但需要额外编码。你可以通过实现类方法`automaticallyNotifiesObserversForKey:` 来控制你子类的属性是否自动通知。

在有些场景下，一个对象的 A 属性 改变可能导致它的 B 属性改变，例如人的名字改变会导致它的全名改变。这种情况可以通过注册依赖 key 来解决。注册依赖 key 则要区分 to-one relationship 和 to-many relationship。

to-one relationship

对于 to-one relationship 有两种方法：一是覆盖 `keyPathsForValuesAffectingValueForKey: `; 二是实现 `keyPathsForValuesAffecting<Key>`。

```
- (NSString *)fullName {
    return [NSString stringWithFormat:@"%@ %@",firstName, lastName];
}

+ (NSSet *)keyPathsForValuesAffectingValueForKey:(NSString *)key {

    NSSet *keyPaths = [super keyPathsForValuesAffectingValueForKey:key];

    if ([key isEqualToString:@"fullName"]) {
        NSArray *affectingKeys = @[@"lastName", @"firstName"];
        keyPaths = [keyPaths setByAddingObjectsFromArray:affectingKeys];
    }

    return keyPaths;
}

+ (NSSet *)keyPathsForValuesAffectingFullName {
    return [NSSet setWithObjects:@"lastName", @"firstName", nil];
}

```

当使用 category 为已有的类添加计算属性时你不能覆盖 `keyPathsForValuesAffectingValueForKey:` 方法，因为你不能在 category 中覆盖方法。在这种情况下，实现对应的 `keyPathsForValuesAffecting<Key>` 类方法来利用这种机制。

to-many relationship

`keyPathsForValuesAffectingValueForKey:` 方法不支持包含 to-many relationship 的 key-path。 例如，假设你有一个 Department 对象拥有一个 to-many relationship(employee)，并且 Employee 有一个 salary 属性。你也许想让 Department 对象有一个 totalSalary 属性，它依赖于所有雇员的薪水。你不能使用 `keyPathsForValuesAffectingTotalSalary` 返回 employees.salary 做成这件事。

这种情况有两种可能的解决方法：

1. 你可以注册父对象(本例中是 Department)作为关联属性所有子对象(本例中是 Employees)的观察者。当子对象添加到关系中或从关系中删除，你必须添加和移除作为观察者的父对象。在 `observeValueForKeyPath:ofObject:change:context:` 方法中更新相应的依赖以响应变化，如以下代码片段所示：

```
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *)context {
    if (context == totalSalaryContext) {
        [self updateTotalSalary];
    }
    else
    // deal with other observations and/or invoke super...
}

- (void)updateTotalSalary {
    [self setTotalSalary:[self valueForKeyPath:@"employees.@sum.salary"]];
}

- (void)setTotalSalary:(NSNumber *)newTotalSalary {

    if (totalSalary != newTotalSalary) {
        [self willChangeValueForKey:@"totalSalary"];
        _totalSalary = newTotalSalary;
        [self didChangeValueForKey:@"totalSalary"];
    }
}

- (NSNumber *)totalSalary {
    return _totalSalary;
}
```

2. 如果你使用 Core Data，你可以把它的父对象作为 managed object context 的观察者注册到通知中心。父对象应该用 key-value observing 类似的方法来响应子对象发出的变化通知。

####KVO 属性为 NSArray 的示例

```
NSMutableArray *array = [self.KVOArray mutableArrayValueForKey:@"KVOArray"];
        
[array addObject:anObject];
```

[KVO Array Demo](https://github.com/DamianSheldon/KeyValueObservingDemo.git)  

Extension:[如何自己动手实现 KVO](http://tech.glowing.com/cn/implement-kvo/)

####Key-Value Observing 实现细节


自动 key-value observing 是使用称为 **isa-swizzling** 的技术实现的。

**isa** 指针，就像它的名字，指向对象维护分发列表的类。分发列表包含类实现方法的指针。

当一对象的属性注册了观察者， **isa** 指针就被修改指向一个中间类而不是真实的类。结果是 **isa** 指针的值没有必要反映实例的真正类。

你应该绝不应该使用 isa 指针来确定类关系。相反，你应该使用 **class** 方法来确定实例对象的类。


#Reference

Key-Value Coding Programming Guide   
Key-Value Observing Programming Guide


