---
layout: post
title: "(翻译)iOS 设计模式"
date: 2014-11-10 08:35:43 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Design Patterns, iOS
description: Introduce to iOS Design Patterns
---
*iOS 设计模式*--你也许听说过这个术语，但是你知道它意味着什么吗？尽管大多数开发者可能同意设计模式非常重要，但关于它的文章并不多，咱们开发者们写代码时并没有花太多注意力放到设计模式上。  

在软件设计中设计模式是常见问题的可复用的解决方法。他们被设计成模板帮助你写出容易理解和复用的代码。他们也帮助你写出低藕合的代码以便你不需要太多争论就能改变或替换你代码的组件。  

如果你刚刚接触设计模式，告诉你些好消息！首先，你已经使用过很多设计模式，这得感谢Cocoa 建立的方式以及你被鼓励使用的最佳实践。其次，这篇教程将让你掌握 iOS 的主要（次要）设计模式，它们在 Cocoa 中很常用。

教程分成多个部分，每部分包含一种设计模式。在每部分，你会看到如下顺序的解释： 
 
* 该设计模式是什么；
* 你为什么应该使用它；
* 如何使用它，以及什么场景合适，使用时需要留意的常见陷阱；

这篇教程中，你将创建一个 Music Library 应用，它会显示你的专辑和它们相关的信息。

在开发应用的过程中，你会慢慢熟悉多数常见的 Cocoa 设计模式：   

* 构造类：单例(Singleton)和 抽象工厂(Abstract Factory);
* 架构类：模型-视图-控制器(MVC), 修饰(Decorator), 适配器(Adapter), 门面(Facade)和合成（Composite);
* 行为类：观察者(Observer), 记忆(Memento), 响应链(Chain of Responsibility)和命令（Command）。

别被误导认为这是篇纯理论的文章；你会在你音乐应用中使用这些应用模式中的大多数。你的应用最终看起来像这样：   

<div style="text-align:center" markdown="1">

<img name="finalapp-180x320" src="/images/finalapp-180x320.png" width="180" height="320">  

</div>

<!-- more -->

###开始吧

下载[启动工程](http://cdn2.raywenderlich.com/wp-content/uploads/2013/07/BlueLibrary-Starter.zip),解压 ZIP 文件内容，然后在 Xcode 中打开。

里面内容不多，仅仅是默认的 ViewController 和一个未实现的简单 HTTP 客户端。

**Note**:你知道吗？当你创建一个新的工程你的代码就已经符合设计模式了。MVC, Delegate, Protocol, Singleton--你免费得到全部！:]

在你深入第一个设计模式之前，你必须创建两个类去持有和显示专辑数据。

导航到**“File\New\File…”**(或简单地按Command+N)。选中**iOS > Cocoa Touch**，然后**Objective-C class**和单击**Next**。设置类名为**Album**并继承**NSObject**。单击**Next**然后**Create**。

打开**Album.h**，然后添加如下属性和方法原型到**@interface** 和 **@end**间: 
   
```
@property (nonatomic, copy, readonly) NSString *title, *artist, *genre, *coverUrl, *year;
 
- (id)initWithTitle:(NSString*)title artist:(NSString*)artist coverUrl:(NSString*)coverUrl year:(NSString*)year;
```

注意到所有的属性都是可读的，因为 Album 对象被创建之后不需要被改变。

方法是对象的初始化方法。当你创建一个新 alum，你将传入 album name, artist, album cover URL 和 year。

现在打开**Album.m**,添加下列代码到**@implementation**和**@end**中间:   

```
- (id)initWithTitle:(NSString*)title artist:(NSString*)artist coverUrl:(NSString*)coverUrl year:(NSString*)year
{
    self = [super init];
    if (self)
    {
        _title = title;
        _artist = artist;
        _coverUrl = coverUrl;
        _year = year;
        _genre = @"Pop";
    }
    return self;
}
```

这里没有什么神奇的；仅仅是一个简单的 init 方法去创建一个新 Album 实例。

再次导航到**File\New\File…**。选择**Cocoa Touch**然后**Objective-C class**，点击**Next**。设置类名为**AlbumView**，但是这次设置它为**UIView**的子类。点击**Next**然后**Create**。

**Note**:如果你发现快捷键更容易用的话，**Command+N**将创建新的文件，**Command+Option+N**将创建新组，**Command+B**将编译你的工程，**Command+R**将运行工程。

打开**AlbumView.h**,然后添加下列方法原型到**@interface**和**@end**的中间。 

```
- (id)initWithFrame:(CGRect)frame albumCover:(NSString*)albumCover;
```

现在打开**AlbumView.m**，用下面的代码替换**@implementation**之后的内容：  

```
@implementation AlbumView
{
    UIImageView *coverImage;
    UIActivityIndicatorView *indicator;
}
 
- (id)initWithFrame:(CGRect)frame albumCover:(NSString*)albumCover
{
    self = [super initWithFrame:frame];
    if (self)
    {
 
        self.backgroundColor = [UIColor blackColor];
        // the coverImage has a 5 pixels margin from its frame
        coverImage = [[UIImageView alloc] initWithFrame:CGRectMake(5, 5, frame.size.width-10, frame.size.height-10)];
        [self addSubview:coverImage];
 
        indicator = [[UIActivityIndicatorView alloc] init];
        indicator.center = self.center;
        indicator.activityIndicatorViewStyle = UIActivityIndicatorViewStyleWhiteLarge;
        [indicator startAnimating];
        [self addSubview:indicator];
    }
    return self;
}
 
@end
```

你注意到的第一件事是这里有个名为 coverImage 的实例变量。变量代表专辑的封面图片。第二个变量是一个指示器，当专辑在下载时它转动指示器。

在初始化方法的实现中，你设置背影为黑色，创建一个与主视图周边有5个点距离的图片视图，创建并添加了一个活动指示器。

**Note**:很奇怪为什么把私有变量定义在实现文件中而不是接口文件中？这是因为 AlbumView 类之外的其他类并不需要知道这些变量的存在，它们仅被类的内部实现使用。如果你是在开发库或框架让其他开发者使用，这个惯例极其重要。

编译你的工程（**Command+B**）确认所有事情都准备就绪。都好了吗？然后准备迎接你的第一个设计模式！:]

###模型--视图--控制器，设计模式之王

<div style="text-align:center" markdown="1">

<img name="mvcking" src="/images/mvcking.png" width="293" height="196">  

</div>

模型--视图--控制器（MVC）是 Cocoa 的一个基石，它毫无疑问是被用得最多的设计模式。它依据应用中类的角色给它们分类，鼓励基于角色简洁地分隔代码。

这三个角色是：

* 模型：持有你应用数据并定义如何操作它们的对象。例如，这个应用中的模型是你的 Album 类。

* 视图：控制模型类的可视显示以及和用户交互的对象;所有的 UIView 和它们的子类基本上都是。在你这个应用中 AlbumView 代表视图。

* 控制器：控制器是中间件，它协调所有的工作。它从模型类访问数据并显示到视图上，监听事件，在需要时操作数据。你能猜到哪个类是你的控制器吗？对，是 ViewController。

你应用中这个设计模式好的实现意味着每个对象都会是三者之一。


视图和模型的通信可以被最佳描述成下图：

<div style="text-align:center" markdown="1">

<img name="mvc0" src="/images/mvc0.png" width="424" height="194">  

</div>

任何数据发生改变模型类便通知控制器，接下来，控制器将数据更新到视图上。视图接收到用户的动作时可以通知控制器，控制器会根据需要更新模型数据或获取任何请求的数据。

你也许会奇怪为什么不拿掉 Controller ,在同一个类中实现视图和模型，这看起来更容易。

这都来源于代码的去藕合和可复用。理想情况下，视图应该完全和模型隔离，这样它可以被不同的模型复用去展示其他的数据。

例如，如果未来你也想添加电影或书本到你的收藏库中，你仍然可以使用相同的 AlbumView 去展示你的电影和图书对象。此外，如果你创建了一个新的对象，它和专辑有些关系，你可以简单地复用你的 Album 类，因为它不依赖任何视图。这就是 MVC 的强大之处！

###如何使用 MVC 设计模式

首先，你需要保证你工程中的类是 Controller，View, Model三者之一；不要混合两个角色的功能到一个类中。你创建的**Album**和**AlbumView**到目前为止都做的很好。

其次，为了确保你习惯这种工作方式，你应该创建三个工程组来持有你的代码，每组对应一个类别。

导航到**File\New\Group**（或按**Command+Option+N**）并命名为Model，用相同的方法创建**View**和**Controller**组。

现在拖拽**Album.h**和**Album.m**到**Model**组。拖拽**AlbumView.h**和**AlbumView.m**到**View**组，最后拖拽**ViewController.h**和**ViewController.m**到**Controller**组。

目前你的工程结构应该看起来像这样：

<div style="text-align:center" markdown="1">

<img name="mvc2-255x320" src="/images/mvc2-255x320.png" width="255" height="320">  

</div>

你的工程已经没有混乱的文件了看起来好多了。你明显能拥有其他的组和类，但这三个类别中的类是应用的核心。

现在你的组件被组织起来了，你需要从其他地方得到你的专辑数据。你将创建一个 API 类用于全部代码的数据管理--这提供了一个机会和你探讨下一个设计模式--单例。

###单例设计模式

单例设计模式确保对于指定的类仅存在一个实例，全局的访问都指向它。它经常使用懒散加载，只有第一次需要时才创建这个实例。

**Note**:Apple大量使用这种方法。例如：**[NSUserDefaults standardUserDefaults]**, **[UIApplication sharedApplication]**, **[UIScreen mainScreen]**, **[NSFileManager defaultManager]** 全都返回单例对象。

你也许会奇怪为什么你要关心一个类是不是只有一个实例，毕竟代码和内存都很便宜，对不对？

有些场景对于类只存在一个实例是有意义的。例如，没有必要存在多个 Logger 实例，除非你想同时输出多个 log 文件。或者来看一个全局配置处理类：像配置文件，对于单个共享资源实现线程安全访问要比在同时可能有很多配置文件修改时容易的多。

###如何使用单例设计模式

看下下面这个图：

<div style="text-align:center" markdown="1">

<img name="singleton" src="/images/singleton.png" width="233" height="152">  

</div>

上图示例了一个 Logger 类，它有一个属性（它就是这个单独的实例），和两个方法： sharedInstance 和 init。

客户端第一次发送 sharedInstance 消息，属性的实例还没被初始化，所以你创建类的一个新的实例，然后返回它的一个引用。

下次调用 sharedInstance，实例会立即返回不用初始化。这个逻辑保证任何时候都只仅存在一个实例。

你将实现这种模式通过创建一个单例类来管理专辑的所有数据。

你将注意到工程里有一个组叫**API**；这是你放所有将为应用提供服务类的地方。在组里用**iOS\Cocoa Touch\Objective-C class**模板创建一个新的类。命名为**LibraryAPI**，设置它是**NSObject**的子类。

打开**LibraryAPI.h**,用如下内容替代它：

```
@interface LibraryAPI : NSObject
 
+ (LibraryAPI*)sharedInstance;
 
@end
```

现在到**LibraryAPI.m**，在**@implentation**之后插入这个方法：

```
+ (LibraryAPI*)sharedInstance
{
    // 1
    static LibraryAPI *_sharedInstance = nil;
 
    // 2
    static dispatch_once_t oncePredicate;
 
    // 3
    dispatch_once(&oncePredicate, ^{
        _sharedInstance = [[LibraryAPI alloc] init];
    });
    return _sharedInstance;
}
```

简短的方法里有不少内容：

 1. 声明了一个静态变量来持有你类的实例，确保它在你的类中是全局可用的。
 
 2. 声明了一个**dispatch_once_t**静态变量，它确保初始化方法只会被执行一次。
 
 3. 使用Grand Central Dispatch (GCD)来执行块，它初始化了一个**LibraryAPI**的实例。这是单例设计模式的要义：类被实例化之后初始化方法就不会被调用了。

下次你调用**sharedInstance**，在**dispatch_once**块中代码就不会被执行了（因为它已经被执行过一次了），你会得到一个之前创建的**LibraryAPI**的实例引用。


Note:想了解更多GCD内容以及它的用法，看下网站中的这两篇教程：[Multithreading and Grand Central Dispatch](http://www.raywenderlich.com/?p=4295)和[How to Use Blocks](http://www.raywenderlich.com/?p=9328)。

你现在有一个单例对象作为入口去管理专辑。我们把它进一步完善，创建一个类来处理你收藏库数据的持久化。

以**iOS\Cocoa Touch\Objective-C class**为模板在API组中创建一个新的类，命名为**PersistencyManager**，设置它为**NSObject**的子类。

打开**PersistencyManager.h**，添加下列引用到文件的顶端:

```
#import "Album.h"
```

然后，添加下列代码到**PersistencyManager.h**中**@interface**后面：

```
- (NSArray*)getAlbums;
- (void)addAlbum:(Album*)album atIndex:(int)index;
- (void)deleteAlbumAtIndex:(int)index;
```

上面是你处理专辑数据的三个方法原型。

打开 PersistencyManager.m，添加下列代码到 @implementation 前面：

```
@interface PersistencyManager () {
    // an array of all albums
    NSMutableArray *albums;
}
```

上面添加了一个类的扩展，它是另一种给类添加私有方法和变量而不会暴露给外部类的方法。这里你声明了一个**NSMutableArray**来持有专辑数据。数组的可变性可以让你很容易添加和删除专辑数据。

现在添加下现代码实现到 PersistencyManager.m 文件中 @implementation 的后面：

```
- (id)init
{
    self = [super init];
    if (self) {
    	// a dummy list of albums
        albums = [NSMutableArray arrayWithArray:
                 @[[[Album alloc] initWithTitle:@"Best of Bowie" artist:@"David Bowie" coverUrl:@"http://www.coversproject.com/static/thumbs/album/album_david%20bowie_best%20of%20bowie.png" year:@"1992"],
                 [[Album alloc] initWithTitle:@"It's My Life" artist:@"No Doubt" coverUrl:@"http://www.coversproject.com/static/thumbs/album/album_no%20doubt_its%20my%20life%20%20bathwater.png" year:@"2003"],
                 [[Album alloc] initWithTitle:@"Nothing Like The Sun" artist:@"Sting" coverUrl:@"http://www.coversproject.com/static/thumbs/album/album_sting_nothing%20like%20the%20sun.png" year:@"1999"],
                 [[Album alloc] initWithTitle:@"Staring at the Sun" artist:@"U2" coverUrl:@"http://www.coversproject.com/static/thumbs/album/album_u2_staring%20at%20the%20sun.png" year:@"2000"],
                 [[Album alloc] initWithTitle:@"American Pie" artist:@"Madonna" coverUrl:@"http://www.coversproject.com/static/thumbs/album/album_madonna_american%20pie.png" year:@"2000"]]];
    }
    return self;
}
```

在 init 中，你用5个样本专辑填充了数组。如果上面的专辑你不喜欢，用你喜欢的音乐替换它们吧. :]

现在添加下面三个方法到**PersistencyManager.m**：

```
- (NSArray*)getAlbums
{
    return albums;
}
 
- (void)addAlbum:(Album*)album atIndex:(int)index
{
    if (albums.count >= index)
    	[albums insertObject:album atIndex:index];
    else
    	[albums addObject:album];
}
 
- (void)deleteAlbumAtIndex:(int)index
{
    [albums removeObjectAtIndex:index];
}
```

这些方法能让你获取，添加，删除专辑。

编译你的工程，确保所有的文件仍然能成功被编译。

到这，你可能会奇怪**PersistencyManager**来自哪里呢？它又不是单例。**LibraryAPI**和**PersistencyManager**的关系，我们在下一节会进一步揭示，你还会看到**门面**设计模式。

###门面设计模式

<div style="text-align:center" markdown="1">

<img name="facade" src="/images/facade.jpg" width="300" height="212">  

</div>

门面设计模式为复杂的子系统提供一个单一的接口。与其把一系列类和它们的API暴露给用户，还不如仅仅暴露给它们一个简单统一的API。

下图解释了这种理念：

<div style="text-align:center" markdown="1">

<img name="facade2-480x241" src="/images/facade2-480x241.png" width="480" height="241">  

</div>

API的使用者们完全感觉不到它后面的复杂。当很多类协作时，这种设计模式十分理想，特别是当它们对用户很复杂或很难理解。

门面模式让使用系统的代码从接口和你隐藏实现的类解藕；它也减少了外部代码对内部子系统工作的依赖。当门面底下的类想改变时，这也很有用，因为门面能保留相同的API，尽管后面的代码已经改变了。

例如，如果有一天你想替换你的后端服务，你不需要去修改使用你 API 的代码。

###如何使用门面设计模式

目前你有**PersistencyManager**保存专辑数据到本地，**HTTPClient**处理远程交互。工程中其他的类不应该意识到这个逻辑的存在。

为了实现这个设计模式，仅**LibraryAPI**应该持有**PersistencyManager**和**HTTPClient**的实例。然后，**LibraryAPI**会对其他的服务暴露一个简单的 API。

**Note**:通常，单例在整个应用的生命周期都存在。你不应该让单例保持大多其他对象的强引用，因为他们直到应用关闭才会被释放。

设计看起来像下面这样：

<div style="text-align:center" markdown="1">

<img name="design-patterns-facade-uml-480x71" src="/images/design-patterns-facade-uml-480x71.png" width="480" height="71">  

</div>

**LibraryAPI**将暴露给其他代码，但是会对应用的其他部分隐藏**HTTPClient**和**PersistencyManager**的复杂。

打开**LibraryAPI.h**，添加下列引用到文件的顶端：

```
#import "Album.h"
```

然后，添加下列方法定义到**LibraryAPI.h**：

```
- (NSArray*)getAlbums;
- (void)addAlbum:(Album*)album atIndex:(int)index;
- (void)deleteAlbumAtIndex:(int)index;
```

现在，这些方法是你会暴露给其他类的。

到LibraryAPI.m中，添加下面两个引用：

```
#import "PersistencyManager.h"
#import "HTTPClient.h"
```

这将是你唯一导入这些类的地方。记住：你的 API 将会是唯一的入口去访问你"复杂"的系统。

现在，通过类的扩展添加些私有变量（在@implementation上面）：

```
@interface LibraryAPI () {
    PersistencyManager *persistencyManager;
    HTTPClient *httpClient;
    BOOL isOnline;
}
 
@end
```

**isOnline**决定任何专辑列表的改变，例如，添加或删除专辑，是否应该被更新到服务器。

你现在需要通过**init**来初始化这些变量。添加下列代码到**LibraryAPI.m**：

```
- (id)init
{
    self = [super init];
    if (self) {
        persistencyManager = [[PersistencyManager alloc] init];
        httpClient = [[HTTPClient alloc] init];
        isOnline = NO;
    }
    return self;
}
```

HTTP Client 最终不会和真实的服务器交互，这里仅仅是为示例门面模式的使用，所以**isOnline**将一直是**NO**。

下一步，添加下面三个方法到**LibraryAPI.m**：

```
- (NSArray*)getAlbums
{
    return [persistencyManager getAlbums];
}
 
- (void)addAlbum:(Album*)album atIndex:(int)index
{
    [persistencyManager addAlbum:album atIndex:index];
    if (isOnline)
    {
        [httpClient postRequest:@"/api/addAlbum" body:[album description]];
    }
}
 
- (void)deleteAlbumAtIndex:(int)index
{
    [persistencyManager deleteAlbumAtIndex:index];
    if (isOnline)
    {
        [httpClient postRequest:@"/api/deleteAlbum" body:[@(index) description]];
    }
}
```

看一眼**addAlbum:atIndex:**。类首先更新本地数据，然后如果是联网的话，更新远程服务器。这是门面的真正力量；当你系统外面添加一个新专辑，它不知道，也不需要知道这底下的复杂。

**Note**:当你为子系统的类设计门面时，记住没任何东西防止客户端直接访问隐藏的类。不要吝啬你的防御代码，不要假设所有客户端按门面相同的方式使用这些类是必须的。

编译并运行你的应用。你会看到像下面这样令人兴奋和无法置信的空的黑色屏幕。

<div style="text-align:center" markdown="1">

<img name="2013-09-01_12-08-44-211x320" src="/images/2013-09-01_12-08-44-211x320.png" width="211" height="320">  

</div>

你将会需要些东西来显示专辑数据到屏幕上--这是你的下个设计模式--修饰，完美的使用场景。

###修饰设计模式

修饰设计模式动态添加行为和能力到一个对象而不需要修改它的代码。它是不同于子类那样通过包装到另一个对象来修改类的行为的方法。

在Objective-C中，这种设计模式有两个很常见的实现：**Category**和**Delegation**。

####Category

Category是一个极其强大的机制，它允许你添加方法到已经存在的类而不需要子类化。新的方法在编译时被添加，可以像扩展类的普通方法一样被执行。它和经典的修饰模式有点不同，因为一个Category不能持有扩展类的实例。

**Note**:除了扩展你自己的类，你还可以添加方法到任意Cocoa拥有的类。

####如何使用Categories

想像这么一个场景，你有一个Album对象，你想让它显示在一个表格视图中:

<div style="text-align:center" markdown="1">

<img name="design-patterns-category1" src="/images/design-patterns-category1.png" width="310" height="188">  

</div>

专辑标题是从哪来的呢？**Album**是一个模型对象，所以它不关心你如何展示数据。你将需要些外部代码来为**Album**类添加该功能，但是不能直接修改类。

你将创建一个 category，这是 Album的扩展；它将定义一个新方法，这个新方法会返回一个让UITableView很容易使用的数据结构。

这个数据结构会看起来像这样：

<div style="text-align:center" markdown="1">

<img name="delegate2-480x67" src="/images/delegate2-480x67.png" width="480" height="67">  

</div>

为了添加**Category**到**Album**，导航到**File\New\File…**，选择**Objective-C category**模板--不要习惯性地选择了**Objective-C class**！输入**TableRepresentation**到**Category**字段，**Album**到**Category on**字段。

Note:你有没注意到新文件的名字？**Album+TableRepresentation**意味着你正在扩展**Album**类。这个惯例很重要，因为它易读并且它防止和你事其他人可能创建的categories冲突。

进入Album+TableRepresentation.h，添加如下方法原型：

```
- (NSDictionary*)tr_tableRepresentation;
```

注意这里的方法名前有个**tr_**，是**category:TableRepresentation**的缩写。再次提醒，像这样的惯例将防止和其他方法冲突！

**Note**:如果你在 category 中声明的方法和源类，或都同一个类其他的category（甚至父类）方法相同，运行时会使用哪个方法实现是未定义的。这种情况在你使用自己拥有类的 category 时很少发生，但是当使用 categories 添加方法到标准的 Cocoa 或 Cocoa Touch 类时能导致严重问题。

进入**Album+TableRepresentation.m**，添加如下方法：

```
- (NSDictionary*)tr_tableRepresentation
{
    return @{@"titles":@[@"Artist", @"Album", @"Genre", @"Year"],
             @"values":@[self.artist, self.title, self.genre, self.year]};
}
```

思考下这种模式在某个时刻有多强大：

* 你正在使用直接来自Album属性。
* 你添加了内容到Album类，但是你并没有子类化它。如果你需要子类化Album，你仍然也可以这么做。
* 这个简单的额外内容让你能返回一个UITableView式的专辑，并没有修改Album的代码。

Apple 在 Foundation 类中大量使用 Categories。打开**NSString.h**看下他们是如何做的。找到 **@interface NSString**，你将会看到总共定义了三个categories：**NSStringExtensionMethods**, **NSExtendedStringPropertyListParsing** 和 **NSStringDeprecated**。Categories 帮助方法组织和分隔到各个部分。

####Delegation

另一个修饰设计模式，Delegation 是一种一个对象的行为依赖或基于另一个对象的机制。例如，当你使用**UITableView**，你必须实现的方法之一是**tableView:numberOfRowsInSection:**。

你不能期望 UITableView 知道你想每个部分有多少行，因为这是应用特定的。因此，计算每个部分有多少行的任务传递给了 UITableView delegate。它允许 UITableView 类独立于它显示的数据。

这里有一个当你创建一个 UITableView 时是如何进行的虚拟场景解释：

<div style="text-align:center" markdown="1">

<img name="delegate-480x252" src="/images/delegate-480x252.png" width="480" height="252">  

</div>

UITableView 对象的工作是显示 table view。但是，最终它将需要一些它没有的信息。然后，它求助于它的 delegates，发送消息询问额外的信息。在 Objective-C 中实现delegate 模式，一个类通过 protocol 可以声明可选和必选的方法。教程稍候全面介绍  protocols。

表面看起来仅仅去继承一个对象然后覆盖必要的方法要简单，但是考虑下你只能继承单一的一个类。如果你想让某个类成为两或多个对象的 delegate，这是不能通过继承实现的。

Note:这是个很重要的模式。Apple 应用这种方法到大多数 UIKit 类：**UITableView**, **UITextView**, **UITextField**, **UIWebView**, **UIAlert**, **UIActionSheet**, **UICollectionView**, **UIPickerView**, **UIGestureRecognizer**, **UIScrollView**。列表还在继续。

未完待续...

###原文

[iOS Design Patterns](http://www.raywenderlich.com/46988/ios-design-patterns)

