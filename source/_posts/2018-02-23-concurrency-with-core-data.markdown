---
layout: post
title: "Concurrency with Core Data"
date: 2018-02-23 16:00:35 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Concurrency, Core Data
description: Concurrency with Core Data
---

平常在项目中没有使用过 Core Data, 因为我觉得它的学习曲线还挺陡峭，整个框架给人的感觉很复杂和笨重，因此一直没有使用它。但是看到喵神这份[上级向的十个 iOS 开发面试题]()中和这份[百度面试]()题中都有涉及到 Core Data 的内容，我想还是有必要好好研究一下它，毕竟它是 Apple 官方的持久化方案，我们可以取其精华，弃其糟粕，另一方面未来我们也可能因为各种原因接手或参与使用 Core Data 的项目。  

这篇文章主要想探讨上面提到的面试题中的两个关于 Core Data 的问题:

1. 你实现过多线程的Core Data么？NSPersistentStoreCoordinator，NSManagedObjectContext和NSManagedObject中的哪些需要在线程中创建或者传递？你是用什么样的策略来实现的？
2. Core Data：中多线程中处理大量数据同步时的操作。

在回答这两个问题之前，我们先看 Apple 是怎么告诉我们使用多线程的 Core Data 的，在最新的(2017-03-27) Core Data Programming Guide 中有一节 Concurrency with Core Data，它没有直接说如何使用多线程，只是说了 managed object context 在多线程中的两种使用模式:

> In Core Data, the managed object context can be used with two concurrency patterns, defined by NSMainQueueConcurrencyType and NSPrivateQueueConcurrencyType.
NSMainQueueConcurrencyType is specifically for use with your application interface and can only be used on the main queue of an application.
The NSPrivateQueueConcurrencyType configuration creates its own queue upon initialization and can be used only on that queue. Because the queue is private and internal to the NSManagedObjectContext instance, it can only be accessed through the performBlock: and the performBlockAndWait: methods.

对于多线程中对象的传递

说实话我通篇看完以后也没搞明白该如何使用多线程的 Core Data，于是我又找了 Apple 提供的多线程的 Core Data 示例代码 [ThreadedCoreData]()，它展示了一种使用多线程的 Core Data 的方法，但是并不能解答如何使用多线程的 Core Data。因为可能还有很多其他的方法，我们要溯本求源，找到问题的关键，问题才能迎刃而解。 于是我又到 [objc 中国]()上查找，里面专门有一个 Core Data 的专题，先看了一遍[导入大数据集]()，它基本上解答了问题2，我们稍候将它总结为答案，同时它还提供了新的线索，[在后台使用 Core Data]()，于是我又看了这篇文章。

这篇文章提到在使用多线程的 Core Data 时，强烈建议先通读 Apple 的官方文档 Concurrency with Core Data，这也是符合学习 iOS 开发新知识的路线的，毕竟所有的知识都源于 Apple，这种方法推荐给大家，而我一开始也是这么做的，这里的问题是 Apple 的文档一直在更新，有的内容在新版本文档中被删除了，那么我们有办法找到旧版本的文档吗？

有的，这里介绍一种方法，虽然 Apple 不提供旧版本的文档，但是有个网址它会定期备份整个互联网上重要的网址，所以我们可以结合文档的修改历史在这里找到旧版本的文档，我们看到在后台使用 Core Data 翻译于 2014/03/22，我们不妨先试下 2014-03-10 这个版本的 Core Data Programming Guide.

这个版本是这么介绍如何使用多线程的 Core Data 的:

> The pattern recommended for concurrent programming with Core Data is thread confinement : each thread must have its own entirely private managed object context.
> There are two possible ways to adopt the pattern:
> 1. Create a separate managed object context for each thread and share a single persistent store coordinator.
> This is the typically-recommended approach.
> 2. Create a separate managed object context and persistent store coordinator for each thread.
> This approach provides for greater concurrency at the expense of greater complexity (particularly if you need to communicate changes between different contexts) and increased memory usage.

个人认为这个版本的介绍更清晰明了，也很容易得出问题的答案：


正如喵神所说面试中的技术问题环节不仅是企业对应聘者技能和积累的考察，也是一个开发者自我检验的好机会。而且面试中的技术问题通常是关于某知识点的难点，即使是我们经常使用的知识，如果我们没有仔细深入地思考可能也答不上来，所以我觉得利用面试题来提高自己的技术水平和加深对某知识的掌握是不错的方法。


