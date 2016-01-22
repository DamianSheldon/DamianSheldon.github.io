---
layout: post
title: "iOS Unit Testing With OCMock"
date: 2016-01-22 14:19:12 +0800
comments: true
categories: [Archives, iOS Development]
keywords: Unit Test, OCMock
discription: iOS Unit Testing With OCMock
---

单元测试是我们保障代码质量的重要手段， Apple 对些也十分重视，这点可以从 Xcode 新建工程时会自动创建单元测试的 Target 看出来。单元测试牵涉的内容很多，这篇文章是目前我对单元测试的理解。  

既然是单元测试，那么什么是单元呢？我没有去考证，但我是这么理解的：面向对象编程范式里一切皆对象，而对象是由实例变量和方法组成，对象之间通过方法互相作用，我们的应用可以看作是一个对象图，对象图上的对象相互作用来实现我们的需求。这么看来我们的单元应该是对象的方法。

对象的方法在工作的时候可能要依赖其他对象，为了去除依赖对象对测试的影响，我们引入 Mock.  

> Mocks are ‘fake’ objects with pre-defined behavior to stand-in for concrete objects during testing. -- HackaZach

引用 Mock 之后，我们在单元测试中如何使用它呢？

> The general recipe for using mocks in unit-tests is:

> 1. Create the mock object
> 2. Specify the expected invocations and return values
> 3. Associate the mock object with the code under test
> 4. Execute the code under test
> 5. Validate that your assertions are correct
<!-- more -->
### Create the mock object

OCMock 中创建 Mock 对象的方法如下：  

| Factory Method | Description |
| -------------- | ----------- |
| +mockForClass: | Create a mock based on the given class |
| +mockForProtocol: | Create a mock based on the given protocol |
| +niceMockForClass: | Create a "nice" mock based on the given class |
| +niceMockForProtocol: | Create a "nice" mock based on the given protocol |
| +partialMockForObject: | Create a mock based on the given object |
| +observerMock: | Create a notification observer (more on this later) |

Mock 对象是只是一个空壳，只能调用预先定义的方法，调用没有预先定义的方法会抛出异常；
Nice Mock 对象也是一个空壳，只是它会忽略没有预先定义的方法不会抛出异常；
Partial Mock 对象是把一个已有的对象转成 Mock，调用没有预先定义的方法，它会把方法传给已存在的对象；  
Observer Mock 对象是用来观察通知的。

### Specify the expected invocations and return values

> Calling either the -expect or -stub method will return an object that you can use to setup your expectations.

* Specify the expected invocations

	1. 不带参数的方法

```

- (void)testInit {
  id mockService = [OCMockObject mockForProtocol:@protocol(AVQuoteService)];
  [[mockService expect] initiateConnection];
  
  AVStockPortfolio *portfolio = [[AVStockPortfolio alloc] initWithService:mockService];
  
  [mockService verify];
}
```
</br>
	2. 带参数的方法

> You can use any of the following OCMArg class methods in place of a real argument when setting up your method expectations:


| OCMArg method | Description |
| ------------- | ----------- |
| +any | Any argument is accepted. |
| +anyPointer | Accepts any pointer |
| +isNil	 | The given argument must be nil |
| +isNotNil	 | The given argument must not be nil |
| +isNotEqual:	 | Given argument is not object-equivalent with expectation |
| +checkWithSelector: | onObject:	Check the argument with the given action/target pair |
| +checkWithBlock: | Check the argument with the given block (OS X 10.6 or iOS 4) |

> OCMock also provides a few handy macros for argument matching:

| Macro | Description |
| ----- | ----------- |
| OCMOCK_ANY()	 | Equivalent to [OCMArg any] |
| OCMOCK_VALUE(value) | A quick way to match a non-object argument |
| CONSTRAINT(selector) | Validate with a given selector on self |
| CONSTRAINTV(selector,value) | 	Validate with a given selector on self and an additional argument |

```
id classMock = OCMClassMock([SomeClass class]);
OCMExpect([classMock someMethodWithArgument:[OCMArg isNotNil]]);

/* run code under test, which is assumed to call someMethod */

OCMVerifyAll(classMock)
```
</br>
	3. 通知

```
- (void)testSellSharesInStock {
  id mock = [OCMockObject observerMock];
  // OCMock adds a custom methods to NSNotificationCenter via a category
  [[NSNotificationCenter defaultCenter] addMockObserver:mock
                                                   name:AVStockSoldNotification
                                                 object:nil];
                                               
  [[mock expect] notificationWithName:AVStockSoldNotification object:[OCMArg any]];

  AVPortfolio *portfolio = [self createPortfolio]; // made-up factory method
  [portfolio sellShares:100 inStock:@"AAPL"];

  [mock verify];
}
```

* Specify the return values

> If methods on your mocks need to return values, you have a variety of methods to call on the object returned by -expect or -stub. 

| Method | Explanation |
| ------ | ----------- |
| -andReturn: | Return the given object |
| -andReturnValue: | Return a non-object value (wrapped in a NSValue) |
| -andThrow: | Throw the given exception |
| -andPost: | 	Post the given notification |
| -andCall:onObject: | Call the selector on the given object |
| -andDo: | Invoke the given block (only on OS X 10.6 or iOS 4) |


### Execution & Validation

* Associate the mock object with the code under test
* Execute the code under test
* Validate that your assertions are correct

```
- (void)testSellSharesInStock {
  id quoteService = [[OCMockObject] mockForProtocol:@protocol(AVQuoteService)];
  [[[quoteService expect] andDo:^(NSInvocation *invocation) {
    // validate arguments, set return value on the invocation object
  }] priceForStock:@"AAPL"];
  
  // Associate the mock object with the code under test
  AVStockPortfolio *portfolio = [[AVStockPortfolio alloc] initWithService:quoteService];
  // Execute the code under test
  [portfolio sellShares:100 inStock:@"AAPL"];
  
  // Validate that your assertions are correct
  
  [quoteService verify];
}
```

## Reference:
[OCMock](http://ocmock.org)  
[Making Fun of Things with OCMock](http://www.archive.alexvollmer.com/posts/2010/06/28/making-fun-of-things-with-ocmock/)  
[OCMock Test Origami](http://hackazach.net/code/2014/03/03/effective-testing-with-ocmock/)  
[IMPROVING IOS UNIT TESTS WITH OCMOCK](http://engineering.aweber.com/improving-ios-unit-tests-with-ocmock/)  

