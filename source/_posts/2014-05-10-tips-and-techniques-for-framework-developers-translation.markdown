---
layout: post
title: "Tips and Techniques for Framework Developers(Translation)"
date: 2014-05-10 09:23:09 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Framework, Development, Tips, Techniques
description: Tips and Techniques for Framework Developers
---

##Initialization
###Class initialization
initialize 类方法是一个一次性，懒散的执行一些代码的地方，它会在类的其他方法被调用之前调用。最典型的应用是设置类的版本号。
动态系统给继承链上每一个类发送initialize方法，即使方法没有实现。因此它可能会被调用多次（例如，一个子类没有实现它。）。通常我们只想初始化代码被执行一次。一种实现的方法是使用dispatch_once();
``` objective-c 
	+ (void)initialize {

    static dispatch_once_t onceToken = 0;

    dispatch_once(&onceToken, ^{

        // the initializing code

    }

}
```  

Note:因为动态系统发送initialize给每一个类，所以它很可能会在子类的上下文中调用—如果子类没有实现initialize，会调用父类的实现。如果在相关类的上下文中有特殊的初始化需求，我们可以进行如下操作而不仅仅是使用dispatch_once();

``` objective-c
if (self == [NSFoo class]) {

    // the initializing code

}
```

你永远不应显式的调用initialize方法。如果你需要触发初始化，调用一些没有副作用的方法，例如：

``` objective-c
[NSImage self];
```

##Designated Initializers

Designated initializer是类的init方法，它会调用父类的init方法。（其他的initializer调用类的init方法）。每个公用类应该至少有一个designed intializer.例如，NSView的initWithFrame:,NSResponder的init方法。这里init方法不是意味着覆盖，不像作为类簇的NSString和其他的抽象类，子类是期望去实现它自己的init方法。

Designated initializer应该标识清晰因为这些信息对于一个想要子类化的人很重要。子类仅需要覆盖designated initializer其他的初始化方法就能正常工作。

当你实现一个框架的类，你经常需要去实现像initWithCoder:和encodeWithCoder:的归档方法。在初始化代码中要小心点，不要去做反归档得到的对象不会发生的事情。如果你的类实现了归档，一个好的实现方法是你的designated initializer和initWithCoder:调用一段相同的程序。
###在初始化过程中检测错误
好的初始化方法应该完全遵循以下步骤去保证正确的错误检测和传递：
调用父类的designated initializer给self重新赋值。
验证返回值是否为nil，它表明父类的初始化发生了一些错误。
如果现在当前类的初始化发生了错误，释放对象并返回nil。

Listing 1你应该怎么做的示例。
Listing 1  在初始化过程中检测错误
``` objective-c
- (id)init {

    self = [super init];  // Call a designated initializer here.

    if (self != nil) {

        // Initialize object  ...

        if (someError) {

            [self release];

            self = nil;

        }

    }

    return self;

	}
```

#版本化和兼容
当你往框架时添加新的类或方法，通常没有必要给新的特性组指定新版本号。开发者通常（或者应该）在Objective-C的运行时进行检查，例如用respondsToSelector:去确定新的特性在给定的系统上是否可用。这些运行时的测试是验证新特性推荐和最动态的方法。

但是，你也可以使用一些技术去确保你的每个新版本框架都被合适的标识和尽可能的兼容早期的版本。

##框架版本
当运行时测试不容易去检测到存在的新特性或修复的bug，你应该提供一些方法给开发者去检查发生的改变。一种实现方法是存储准确的框架版本号并让开发者能访问：
在版本号下编写更改文档。
设置框架的当前版本并让它全局可读取。你也许会把版本号存放在框架的信息属性列表（Info.plist）中然后访问它。

##加上归档键
如果你框架的对象需要被写到nib文件中，它们必须能够归档它们自己。你也要使用文件数据归档机制去归档任何文档。
你应该考虑关于归档的以下问题：
如果归档的key丢失了，从它们中取值会返回nil, NULL, NO, 0或者0.0,这得看你取值的类型。判断它返回的值可以减少你写入的数据。另外，你可以查出键是否被写入归档中。
编码和解码方法都能做些事情去保证向后兼容。例如，新版本类的编码方法可能用键写新的值，但是仍然写到旧的地方，这样旧类仍能理解对象。另外，解码方法可能想用一些合理的方法处理丢失值去维护未来版本的灵活性。

框架类归档键的推荐命名惯例是加上框架中其他API元素使用的前缀然后加上实例变量的名字。仅仅保证名字不会和任何父类或子类冲突。

如果你有一个工具函数它写出基本的数据类型（换句话说，非对象值），确保使用唯一的键。例如，你有一个名为archiveRect的程序，它应该带一个键的参数去归档一个矩形，无论使用与否；或者，如果你写出多个值（例如，4个浮点数），它应该追加唯一的bits给提供的key.

归档位段对于编译器和大小端依赖时很危险。你应该只在出于性能原因，很多bits需要多次写出时才归档它们。查看”Bitfields”获得更多建议。

#异常和错误

大多数Cocoa框架方法并不强制开发者捕获和处理异常。这是因为正常部分的执行不会抛出异常，除非运行和用户错误，通常它是不用来交流的。这些错误的例子包括：
文件没有找到
没有这个用户
应用程序尝试打开错误类型的文件

但是，Cocoa不会抛出异常去暗示如下的编程或者逻辑错误：

数组索引越界
尝试改变不可变对象
错误的参数类型

所谓异常是开发者将在测试期间捕获上述错误然后在传给应用之前解决它们。因此应用没有必要在运行时处理惯常。如果异常抛出而应用的各个部分都没有捕获它，通常最上层的默认处理方法会捕获它然后报告异常，之后继续执行。开发者可以选择替换默认的异常捕获，如给出更详细的错误信息，提供保证数据的机会或者退出应用。

错误是Cocoa框架与其他软件库另一个不同的地方。Cocoa方法通常不会返回错误代码。当有一个合理或者像错误的原因，方法会简单的依靠布尔或对象（nil/non-nil）返回值测试；返回NO或nil的原因会在文档中说明。你不应该在运行时使用错误代码标明程序错误需要处理，而应使用抛出异常或者简单打印错误来替代。

例如，NSDictionary的 objectForKey:方法返回找到的对象,如果对象没有找到则返回nil。NSArray的objectAtIndex:方法永远不能返回nil(除非覆盖通用的语言惯用像给nil发送消息返回nil)，因为NSArray对象不能存储nil值，而且在定义上任何越界访问都量程序错误应该抛出异常。许多初始化方法会返回nil当对象不能被 提供的参数初始化时。

在一些小众的情况下会有一些方法有对许多特定错误代码的合理需求，应该通过想着参数指定他们，返回错误代码，或者本地化错误字符串，或者天王终点其他错误描述信息。例如，你可能把错误作为一个NSError返回；查看NSError.h头文件了解更多细节。这个参数需要额外提供不像BOOL或nil是直接返回的。方法也应该遵守这样一个惯例，通过引用的参数是可选的，并且如果发送者不关心错误应该允许传递NULL作为error-code的参数。

#框架数据
你处理框架数据的方式会影响性能，跨平台兼容和其他方面。这一部分讨论涉及框架数据的技术。

##常量数据

因为性能的原因，尽可能的把常量标记为框架数据是推荐的做法，因为这样可以减少Mach-O二进制文件__DATA段的大小。全局和静态变量不是const，它们在__DATA段的__DATA部分。这种类型的数据会占用内存，当运行的应用使用了这类框架。虽然额外的500字节（例如）也许不是太糟，它可能造成需要许多页—-每个应用额外占用4KB.

你应该把任何常量数据都标记为const.如果没有char*指针在块中，这会导致数据被 放在_TEXT段（这成了真正的常量）。否则它会存在_DATA段但不允许写操作（unless prebinding is not done or is violated by having to slide the binary at load time。）。

你应该初始化静态变量保证它们被合并进_DATA段的_data部分，而不是在_bss部分。如果没有明显的值用作初始化，使用0,NULL, 0.0或任何合适的值。

##位段
位段使用有符号的值，特别是一位的位段，如果代码假设值是布尔类型可能导致未定义行为。一位形式的位段应该总是无符号的。因为它只存像0和-1（依赖编译器实现）这样的唯一的值，拿这样一个位段与1想比较结果是false.例如，如果你的你代码遇到以下情况：


``` objective-c
BOOL isAttachment:1;

int startTracking:1;

```

你应该把类型改为unsigned int.

位段的其他问题是归档。通常，你不应该以它自身的形式把位段写到硬盘或者归档文件，因为当从其他的架构或者编译器读取时可能会不一样。

##内存分配

在框架代码中，如果你能做到不一起分配内存是最好的.如果因为某些原因需要临时的缓存区，通常使用栈的缓存区经分配一个缓存区要好。但是，栈的大小有限制（通常总共是512kb)，所以决定使用栈还得考虑函数和缓存的大小。通常如果缓冲的大小是1000字节（或MAXPATHLEN）或更少，使用栈是可接受的。

如果缓冲的大小超过了栈的缓冲大小，就要使用malloc生成的缓冲了。

Listing 2 给出示例代码片段。

Listing 2  Allocation using both stack and malloc’ed buffer
``` objective-c
	#define STACKBUFSIZE (1000 / sizeof(YourElementType))

	 YourElementType stackBuffer[STACKBUFSIZE];

 	YourElementType *buf = stackBuffer;

 	int capacity = STACKBUFSIZE;  // In terms of 	YourElementType

	 int numElements = 0;  // In terms of YourElementType


	while (1) {

    if (numElements > capacity) {  // Need more room

        int newCapacity = capacity * 2;  // Or whatever your growth algorithm is

        if (buf == stackBuffer) {  // Previously using stack; switch to allocated memory

            buf = malloc(newCapacity * sizeof(YourElementType));

            memmove(buf, stackBuffer, capacity * sizeof(YourElementType));

        } else {  // Was already using malloc; simply realloc

            buf = realloc(buf, newCapacity * sizeof(YourElementType));

        }

        capacity = newCapacity;

    }

    // ... use buf; increment numElements ...

  	}

  	// ...

  	if (buf != stackBuffer) free(buf);

```

#对象比较

你应该意识到通常对象的比较方法isEqual:和相关对象类型的比较方法，如isEqualToString:有重大差别. isEqual方法允许传入任意对象作为参数，如果对象不是相同的对象则返回NO.像isEqualToString:和isEqualToArray:方法，经常假定参数是指定的类型（它经常是方法的接收者）。它们因此不做类型检查，因而它们更快，不过这并不安全。对于需要从外部获取的值，例如，应用的信息属性列表或偏好，推荐使用isEqual:,因为它们安全；当类型是知道的时候，使用isEqualToString:替代。

关于isEqual:更深的点是它连接到hash方法。对于放到基于hash的Cocoa集合如NSDictionary或NSSet中的对象，f[A isEqual:B] == YES 和[A hash] == [B hash]的效果是一样的。因此，如果你覆盖isEqual:,那么你也应该覆盖hash来确保这个不变关系。isEqual方法默认会查找指向每个对象的指针地址，hash返回一个基于每个对象地址的hash值，因此它们的关系是不变的。