---
layout: post
title: "Web 面试题汇总(三)"
date: 2018-09-13 10:20:22 +0800
comments: true
categories: [Archives, Web Development]
keywords: js, web, interview
description: Web 面试题汇总
---

###1.介绍js的基本数据类型。
A:

* Six data types that are primitives:
	* Boolean. true and false.
	* null. A special keyword denoting a null value. Because JavaScript is case-sensitive, null is not the same as Null, NULL, or any other variant.
	* undefined. A top-level property whose value is not defined.
	* Number. An integer or floating point number. For example: 42 or 3.14159.
	* String. A sequence of characters that represent a text value. For example:  "Howdy"
	* Symbol (new in ECMAScript 2015). A data type whose instances are unique and immutable.
* and Object


###2.介绍js有哪些内置对象？

###3.说几条写JavaScript的基本规范？
A:

* 总是使用var声明变量
* 行末别忘了分号
* 使用标准而不是不标准的功能
* 使用驼峰命名（如：camelCaseNames）以及大写的常量（如：UPPERCASE），避免使用const关键字，因为IE不支持
* 使用命名空间技术
* 避免eval()除非反序列化（奇怪的是JSON解析并未提及）
* 避免对象上使用with(), 数组使用for in
* 使用对象以及数组字面量而不是更冗长的声明
* 要知道truthy和falsy规则
* JavaScript资源中不使用IE条件注释
* 不修改内置对象的原型——这会让人颜面扫地，因为这是让JavaScript更加强大的功能之一，但你知道这会导致问题
* 小心使用闭包以及不要循环引用
* 同样，小心使用”this”

Reference:[翻译：谷歌HTML、CSS和JavaScript风格规范](https://www.zhangxinxu.com/wordpress/2012/07/google-html-css-javascript-style-guides/)  

###4.JavaScript原型，原型链 ? 有什么特点？
A:

* 原型：每个对象都有一个原型对象，它充当继承方法和属性来源的模板对象。  
* 原型链：对象的原型对象也可能有原型，依此类推，便形成原型链。  
* 特点：强大而灵活。
<!--more-->
###5.JavaScript有几种类型的值？，你能画一下他们的内存图吗？
A:

* 栈类型：Undefined，Null，Boolean，Number、String
* 堆类型：对象、数组和函数

###6.如何将字符串转化为数字，例如'12.3b'?
A:`parseFloat('12.3b');`

###7.如何将浮点数点左边的数每三位添加一个逗号，如12000000.11转化为12,000,000.11?
A:

* 方法一：使用正则表达式, `String(Number).replace(/(\d)(?=(\d{3})+$)/g, "$1,");`
* 方法二：使用toLocaleString()方法, Number.toLocaleString('en-US');

Reference:[请使用千位分隔符(逗号)表示web网页中的大数字](https://www.zhangxinxu.com/wordpress/2017/09/web-page-comma-number/)  

###8.如何实现数组的随机排序？
A:Fisher–Yates Shuffle,

```
function shuffle(array) {
  var m = array.length, t, i;

  // While there remain elements to shuffle…
  while (m) {

    // Pick a remaining element…
    i = Math.floor(Math.random() * m--);

    // And swap it with the current element.
    t = array[m];
    array[m] = array[i];
    array[i] = t;
  }

  return array;
}
```

Reference:[Fisher–Yates Shuffle](https://bost.ocks.org/mike/shuffle/)  
[数组的完全随机排列](https://www.h5jun.com/post/array-shuffle.html) 

###9.Javascript如何实现继承？
A:Javascript 使用原型链实现继承。

###10.JavaScript继承的几种实现方式？
A:

1. 原型链继承
2. 构造继承
3. 实例继承
4. 拷贝继承
5. 组合继承

```
// Base animal class
function Animal(name)
{
    // Property
    this.name = name || 'Animal';

    // Instance method
    this.sleep = function() {
        console.log(this.name + ' is sleeping');
    };
}

// Prototype method
Animal.prototype.eat = function(food) {
    console.log(this.name + ' is eating ' + food);
};

// 1. 原型链继承
function Cat() {

}

Cat.prototype = new Animal();
Cat.prototype.name = 'tom';

var cat = new Cat();
console.log('cat name ' + cat.name);
cat.sleep();

// 2. 构造继承

function Dog(name) {
    Animal.call(this);

    this.name = name || 'snoopy';
}

var dog = new Dog();
console.log('dog name ' + dog.name);
dog.sleep();

// 3. 实例继承

function Pig(name) {
    var instance = new Animal();
    instance.name = name || 'Penny';

    return instance;
}

var pig = new Pig();
console.log('pig name ' + pig.name);
pig.sleep();

// 4. 拷贝继承

function Bird(name) {
    var animal = new Animal();

    for (var p in animal) {
        Bird.prototype[p] = animal[p];
    }
    Bird.prototype.name = name || 'kiwi';
}

var bird = new Bird();
console.log('bird name ' + bird.name);
bird.sleep();

// 5. 组合继承

function Horse(name) {
    Animal.call(this);
    this.name = name || 'Chitu';
}
Horse.prototype = new Animal();
Horse.prototype.constructor = Horse;

var horse = new Horse();
console.log('horse name ' + horse.name);
horse.sleep();
```

Reference:[JS实现继承的几种方式](http://www.cnblogs.com/humin/p/4556820.html)  

###11.javascript创建对象的几种方式？

###12.Javascript作用链域?

###13.谈谈This对象的理解。

###14.eval是做什么的？

###15.什么是window对象? 什么是document对象?

###16.null，undefined 的区别？

###17.写一个通用的事件侦听器函数。

###18.["1", "2", "3"].map(parseInt) 答案是多少？

###19.事件是？IE与火狐的事件机制有什么区别？ 如何阻止冒泡？

###20.什么是闭包（closure），为什么要用它？

###21.javascript 代码中的"use strict";是什么意思 ? 使用它区别是什么？

###22.如何判断一个对象是否属于某个类？

###23.new操作符具体干了什么呢?

###24.用原生JavaScript的实现过什么功能吗？

###25.Javascript中，有一个函数，执行时对象查找时，永远不会去查找原型，这个函数是？

###26.JSON 的了解？

###27.`[].forEach.call($$("*"),function(a){a.style.outline="1px solid #"+(~~(Math.random()*(1<<24))).toString(16)})` 能解释一下这段代码的意思吗？

###28.js延迟加载的方式有哪些？

###29.Ajax 是什么? 如何创建一个Ajax？

###30.Ajax 解决浏览器缓存问题？

###31.同步和异步的区别?

###32.如何解决跨域问题?

###33.页面编码和被请求的资源编码如果不一致如何处理？

###34.服务器代理转发时，该如何处理cookie？

###35.模块化开发怎么做?

###36.AMD（Modules/Asynchronous-Definition）、CMD（Common Module Definition）规范区别？

###37.requireJS的核心原理是什么？（如何动态加载的？如何避免多次加载的？如何 缓存的？）

###38.JS模块加载器的轮子怎么造，也就是如何实现一个模块加载器？

###39.谈一谈你对ECMAScript6的了解？

###40.ECMAScript6 怎么写class么，为什么会出现class这种东西?

###41.异步加载JS的方式有哪些？

###42.documen.write和 innerHTML的区别?

###43.DOM操作——怎样添加、移除、移动、复制、创建和查找节点?

###44.`.call()` 和 `.apply()` 的区别？

###45.数组和对象有哪些原生方法，列举一下？

###46.JS 怎么实现一个类?怎么实例化这个类?

###47.JavaScript中的作用域与变量声明提升？

###48.如何编写高性能的Javascript？

###49.那些操作会造成内存泄漏？

###50.JQuery的源码看过吗？能不能简单概况一下它的实现原理？

###51.jQuery.fn的init方法返回的this指的是什么对象？为什么要返回this？

###52.jquery中如何将数组转化为json字符串，然后再转化回来？

###53.jQuery 的属性拷贝(extend)的实现原理是什么，如何实现深拷贝？

###54.jquery.extend 与 jquery.fn.extend的区别？

###55.jQuery 的队列是如何实现的？队列可以用在哪些地方？

###56.谈一下Jquery中的bind(),live(),delegate(),on()的区别？

###57.JQuery一个对象可以同时绑定多个事件，这是如何实现的？

###58.是否知道自定义事件。jQuery里的fire函数是什么意思，什么时候用？

###59.jQuery 是通过哪个方法和 Sizzle 选择器结合的？（jQuery.fn.find()进入Sizzle）

###60.针对 jQuery性能的优化方法？

###61.Jquery与jQuery UI 有啥区别？

###62.JQuery的源码看过吗？能不能简单说一下它的实现原理？

###63.jquery 中如何将数组转化为json字符串，然后再转化回来？

###64.jQuery和Zepto的区别？各自的使用场景？

###65.针对 jQuery 的优化方法？

###66.Zepto的点透问题如何解决？

###67.需求：实现一个页面操作不会整页刷新的网站，并且能在浏览器前进、后退时正确响应。给出你的技术实现方案？

###68.如何判断当前脚本运行在浏览器还是node环境中？

###69.移动端最小触控区域是多大？

###70.jQuery 的 slideUp动画 ，如果目标元素是被外部事件驱动, 当鼠标快速地连续触发外部元素事件, 动画会滞后的反复执行，该如何处理呢?

###71.把 Script 标签 放在页面的最底部的body封闭之前 和封闭之后有什么区别？浏览器会如何解析它们？

###72.移动端的点击事件的有延迟，时间是多久，为什么会有？ 怎么解决这个延时？（click 有 300ms 延迟,为了实现safari的双击事件的设计，浏览器要知道你是不是要双击操作。）

###73.解释JavaScript中的作用域与变量声明提升？

###74.Node.js的适用场景？

###75.什么是“前端路由”?什么时候适合使用“前端路由”? “前端路由”有哪些优点和缺点?

###76.检测浏览器版本版本有哪些方式？

###77.What is a Polyfill?

###78.做的项目中，有没有用过或自己实现一些 polyfill 方案（兼容性处理方案）？

###79.我们给一个dom同时绑定两个点击事件，一个用捕获，一个用冒泡。会执行几次事件，会先执行冒泡还是捕获？

###80.使用JS实现获取文件扩展名？

###81.Webpack热更新实现原理?

###82.请介绍一下JS之事件节流？

###83.什么是JS的函数防抖？

