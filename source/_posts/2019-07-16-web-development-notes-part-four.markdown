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


