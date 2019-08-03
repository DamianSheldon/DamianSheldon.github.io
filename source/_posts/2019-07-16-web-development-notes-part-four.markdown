---
layout: post
title: "Web 开发问题汇总(四)"
date: 2019-07-16 17:24:01 +0800
comments: true
categories: [Archives, Web Development]
keywords: bfc
description: Noting problems encounter during web development, every fifteen problem produce a blog, this is the fourth.
---

###1.如何实现如下布局：两个元素A和B，其中A的宽度为包裹其内容，B则占用剩余的宽度？
A:A 元素利用 float 的包裹性，B 元素则利用 BFC。

```
<div class="container">
    <div class="right"></div>
    <div class="left"></div>
</div>

.container {
    width:600px;
    height:200px;
    border:1px solid;
}
.left {
    width:auto;
    height:200px;
    background:red;
    overflow:hidden;
}
.right {
    height:200px;
    width:200px;
    background:blue;
    float:left;
}
```

Reference:[Expand a div to fill the remaining width](https://stackoverflow.com/questions/1260122/expand-a-div-to-fill-the-remaining-width)  

###2.Meaning of ~ in import of scss files
A:

> From documentation on a `sass-loader#imports` project,  

> > webpack provides an advanced mechanism to resolve files. The sass-loader uses node-sass' custom importer feature to pass all queries to the webpack resolving engine. Thus you can import your Sass modules from node_modules. Just prepend them with a ~ to tell webpack that this is not a relative import  

> So if you have a file named foo.css and a module foo then you would use ~ if you want to include the module.

Reference:[Meaning of ~ in import of scss files](https://stackoverflow.com/questions/38880187/meaning-of-in-import-of-scss-files)  

<!--more-->

###3.为什么创建 Observable 时传入的函数签名不需要包含 this? 示例如下:

```
// API Document:
constructor(subscribe?: (this: Observable<T>, subscriber: Subscriber<T>) => TeardownLogic)

// Usage code:
const observable = new Observable(subscriber => {
  subscriber.next(1);
  subscriber.next(2);
  subscriber.next(3);
  setTimeout(() => {
    subscriber.next(4);
    subscriber.complete();
  }, 1000);
});
```

A:Here this is a special syntax in Typescript. It specifies the type of the "this" the function expects. So here it just means it should be called by a Observable with the same Type T as the Subscriber.  

It comes first in the parameter list. It is a fake parameter, and should invoked without it.  

More info can be found [Here](https://github.com/Microsoft/TypeScript/wiki/What's-new-in-TypeScript#specifying-the-type-of-this-for-functions).  

Reference:[Confused by Rxjs Observable constructor and 'this' argument](https://stackoverflow.com/questions/54886652/confused-by-rxjs-observable-constructor-and-this-argument)  

