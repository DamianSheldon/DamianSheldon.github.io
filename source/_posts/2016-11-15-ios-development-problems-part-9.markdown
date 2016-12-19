---
layout: post
title: "iOS 开发问题汇总(九)"
date: 2016-11-15 14:49:17 +0800
comments: true
categories: [Archives, iOS Development] 
keywords: iOS, Swift
discription: 
---
###1.Curried functions in Swift
A:There’s a difference between self.methodname (which you are using), and Classname.methodname.

The former, when called within a class’s method, will give you a function bound with that class instance. So if you call it, it will be called on that instance.

The latter gives you a curried function that takes as an argument any instance of Classname, and returns a new function that is bound to that instance. At this point, that function is like the first case (only you can bind it to any instance you like).

Here’s an example to try and show that a bit better:

```
class C {
    private let _msg: String
        init(msg: String) { _msg = msg }

    func c_print() { print(_msg) }

    func getPrinter() -> ()->() { return self.c_print }
}

let c = C(msg: "woo-hoo")
let f = c.getPrinter()
// f is of type (())->()
f() // prints "woo-hoo"

let d = C(msg: "way-hey")

let g = C.c_print
// g is of type (C) -> (()) -> (),
// you need to feed it a C:
g(c)() // prints "woo-hoo"
g(d)() // prints "way-hey"

// instead of calling immediately,
// you could store the return of g:
let h = g(c)
// at this point, f and h amount to the same thing:
// h is of type (())->()
h() // prints "woo-hoo"
```
Reference:[Curried functions in SWIFT](http://stackoverflow.com/questions/27644702/curried-functions-in-swift)

###2.NSLog on devices in iOS 10 / Xcode 8 will truncate.
A:A temporary solution, just redefine all NSLOG to printf in a global header file.

```
#define NSLog(FORMAT, ...) printf("%s\n", [[NSString stringWithFormat:FORMAT, ##__VA_ARGS__] UTF8String]);
```

Reference:[NSLog on devices in iOS 10 / Xcode 8 seems to truncate? Why？](http://stackoverflow.com/questions/39584707/nslog-on-devices-in-ios-10-xcode-8-seems-to-truncate-why)  

###3.UIImagePickerViewController is black doesn't work for selecting photo or taking picture.
A:I instance it via `UIImagePickerViewController(nibName:nil, boundle:nil)` that doesn't work, and change to `UIImagePickerViewController()` it works.

###4.How to create dispatch queue in Swift?
A:

```
// Concurrent dispatch queue
let concurrentQueue = DispatchQueue(label: "queuename", attributes:.concurrent)

// Serial dispatch queue
let serialQueue = DispatchQueue(label: "queuename")
```

Reference:[How to create dispatch queue in Swift 3](http://stackoverflow.com/questions/37805885/how-to-create-dispatch-queue-in-swift-3)
