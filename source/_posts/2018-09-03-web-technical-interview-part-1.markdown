---
layout: post
title: "Web 面试题汇总(一)"
date: 2018-09-03 17:26:10 +0800
comments: true
categories: [Archives, Web Development]
keywords: html, web, interview
description: Web 面试题汇总
---

###1.Doctype作用？标准模式与兼容模式各有什么区别?
A: 告知浏览器的解析器用什么文档标准解析这个文档。DOCTYPE不存在或格式不正确会导致文档以兼容模式呈现。


###2.HTML5 为什么只需要写 <!DOCTYPE HTML>？
A: HTML5 不基于 SGML，因此不需要对DTD进行引用，但是需要doctype来规范浏览器的行为（让浏览器按照它们应该的方式来运行；而HTML4.01基于SGML,所以需要对DTD进行引用，才能告知浏览器文档所使用的文档类型。

###3.行内元素有哪些？块级元素有哪些？ 空(void)元素有那些？
A:

* 行内元素有：a b span img input select strong（强调的语气）
* 块级元素有：div ul ol li dl dt dd h1 h2 h3 h4…p
* 常见的空元素：br hr img input link meta
* 鲜为人知的空元素：area base col command embed keygen param source track wbr

###4.页面导入样式时，使用link和@import有什么区别？
A:

* link属于XHTML标签，除了加载CSS外，还能用于定义RSS, 定义rel连接属性等作用；而@import是CSS提供的，只能用于加载CSS;
* 页面被加载的时，link会同时被加载，而@import引用的CSS会等到页面被加载完再加载;
* import是CSS2.1 提出的，只在IE5以上才能被识别，而link是XHTML标签，无兼容问题;
* link支持使用js控制DOM去改变样式，而@import不支持;


###5.介绍一下你对浏览器内核的理解？

###6.简述一下你对HTML语义化的理解？
A:用正确的标签做正确的事情。

* html语义化让页面的内容结构化，结构更清晰，便于对浏览器、搜索引擎解析;
* 即使在没有样式CSS情况下也以一种文档格式显示，并且是容易阅读的;搜索引擎的爬虫也依赖于HTML标记
* 确定上下文和各个关键字的权重，利于SEO;
* 使阅读源代码的人对网站更容易将网站分块，便于阅读维护理解。

###7.HTML5的离线储存怎么使用，工作原理能不能解释一下？

###8.浏览器是怎么对HTML5的离线储存资源进行管理和加载的呢？
<!--more-->
###9.请描述一下 cookies，sessionStorage 和 localStorage 的区别？

###10.iframe有那些缺点？
A:   

* iframe会阻塞主页面的Onload事件；
* 搜索引擎的检索程序无法解读这种页面，不利于SEO；
* iframe和主页面共享连接池，而浏览器对相同域的连接有限制，所以会影响页面的并行加载。

使用iframe之前需要考虑这两个缺点。如果需要使用iframe，最好是通过javascript 动态给iframe添加src属性值，这样可以绕开以上两个问题。

###11.HTML5的form如何关闭自动完成功能？
A: 给不想要提示的 form 或某个 input 设置为 autocomplete=off。

###12.如何实现浏览器内多个标签页之间的通信?

###13.webSocket如何兼容低浏览器？

###14.页面可见性（Page Visibility API） 可以有哪些用途？
A: 通过 visibilityState 的值检测页面当前是否可见，以及打开网页的时间等;在页面被切换到其他后台进程的时候，自动暂停音乐或视频的播放；

###15.如何在页面上实现一个圆形的可点击区域？
A:

1. map+area或者svg
2. border-radius
3. 纯js实现 需要求一个点在不在圆上简单算法、获取鼠标坐标等等

###16.实现不使用 border 画出1px高的线，在不同浏览器的标准模式与怪异模式下都能保持一致的效果。
A:

{% codeblock html %}

	<div style="height:1px;overflow:hidden;background:red"></div>
{% endcodeblock %}

###17.网页验证码是干嘛的，是为了解决什么安全问题?
A:区分用户是计算机还是人。

可以防止恶意破解密码、刷票、论坛灌水；有效防止黑客对某一个特定注册用户用暴力破解方式进行不断的登陆尝试。

###18.title与h1的区别、b与strong的区别、i与em的区别？
A:

* title属性没有明确意义只表示是个标题，h1则表示层次明确的标题，对页面信息的抓取也有很大的影响。  
* b 元素用来强调文本，却不表示该文本更重要；strong 元素表示内容更重要。
* i 内容展示为斜体，em表示强调的文本。

###19.Label的作用是什么？是怎么用的？
A:label标签来定义表单控制间的关系,当用户选择该标签时，浏览器会自动将焦点转到和标签相关的表单控件上。

{% codeblock html %}

	<label for="Name">Number:</label>
	<input type="text" name="Name" id="Name"/>
	
	<label>Date:<input type="text" name="B"/></label>
{% endcodeblock %}

##Reference:

* [前端开发面试题](https://github.com/markyun/My-blog/tree/master/Front-end-Developer-Questions/Questions-and-Answers)  

