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

###4.How to make a div take the remaining height?
A:

1. Flex
2. Absolute Positioning
3. Table
4. CSS3 cacl

Reference:[How to make a div take the remaining height](https://www.whitebyte.info/programming/css/how-to-make-a-div-take-the-remaining-height)  

###5.Dynamic modal height based on content in Ionic4?
A:

{% codeblock %}
// Modal page template:
<div class="inner-content">
	<!-- Construct your view hierarchy here-->
</div>

// Present method:
async presentAutoHeightModal() {
	const modal = await this.modalController.create({
      component: AutoHeightModalPage,
      cssClass: 'auto-height'
    });

    return await modal.present();
}

// app.scss
ion-modal.auto-height {
    flex-direction: column;
    justify-content: flex-end;
    --height: auto;
    
    .ion-page {
        position: relative;
        display: block;
        contain: content;

        .inner-content {
            max-height: 80vh;
            overflow: auto;
            padding: 10px;
        }
    }
}
{% endcodeblock %}

###6.Invalid Host Header
A:通过外网访问本地的 ionic 开发服务时提示 Invalid Host Header，原因是 angular 默认会对 host 进行验证，所以我们可以通过传入 `--disableHostCheck=true` 来关闭验证，由于是 ionic 工程，我们传入的参数需要在 `--` 之后，也就是类似如下命令: `ionic serve --port 8080 -- --disableHostCheck=true`.

> ionic serve uses the Angular CLI. Use ng serve --help to list all Angular CLI options for serving your app. See the ng serve docs[1] for explanations. Options not listed below are considered advanced and can be passed to the Angular CLI using the -- separator after the Ionic CLI arguments.

Reference:[Ionic 4 angular - Invalid Host header error](https://community.c9.io/t/ionic-4-angular-invalid-host-header-error/25526)  

Reference:  

* [Dynamic modal height based on content in Ionic v4](https://forum.ionicframework.com/t/dynamic-modal-height-based-on-content-in-ionic-v4/139595)  
* [resize modal based on content](https://github.com/ionic-team/ionic/issues/16852)  

###7.error: illegal character: '\ufeff' in java
A:You probably have a Byte Order Marker (BOM) at the start of the file. You can use sed to remove it:  

```
sed '1s/^.//' infile >> outfile
```

Reference:  

* [error: illegal character: '\ufeff' in java](https://stackoverflow.com/questions/45697794/error-illegal-character-ufeff-in-java/45698146)  

###8.Why super.clone can downcast to subclass?  
A:

```
The method clone for class Object performs a specific cloning operation. First, if the class of this object does not implement the interface Cloneable, then a CloneNotSupportedException is thrown. Note that all arrays are considered to implement the interface Cloneable and that the return type of the clone method of an array type T[] is T[] where T is any reference or primitive type. Otherwise, this method creates a new instance of the class of this object and initializes all its fields with exactly the contents of the corresponding fields of this object, as if by assignment; the contents of the fields are not themselves cloned. Thus, this method performs a "shallow copy" of this object, not a "deep copy" operation.
```

Reference:  

* [Java - downcast in clone](https://stackoverflow.com/questions/19047248/java-downcast-in-clone)  

###9.How to generate MetaModel classes with Spring Data JPA in eclipse?
A: 

1. add maven denpendency  

```
<dependency>
	<groupId>org.hibernate</groupId>
	<artifactId>hibernate-jpamodelgen</artifactId>
</dependency>
```

2. Maven > Update project

###10.Use webroot plugin to obtain certificates
A:

```
$ certbot-auto certonly --webroot --webroot-path /var/lib/tomcat8/webapps/ROOT -d www.tenneshop.com --webroot-path /var/lib/tomcat8/apiapps/ROOT -d api.tenneshop.com
```

Reference:  
* [User Guide](https://certbot.eff.org/docs/using.html?highlight=webroot%20path#webroot)  

###11.flex 布局下怎样实现text-overflow: ellipsis 效果?
A:

{% codeblock %}
<div class="flex-parent">

   <div class="flex-child long-and-truncated">
       1. This is a long string that is OK to truncate please and thank you
   </div>

   <div class="flex-child short-and-fixed">
       <div></div>
       <div></div>
       <div></div>
   </div>

</div>

<div class="flex-parent has-child">

   <div class="flex-child long-and-truncated-with-child">
       <h2>2. This is a long string that is OK to truncate please and thank you</h2>
   </div>

   <div class="flex-child short-and-fixed">
       <div></div>
       <div></div>
       <div></div>
   </div>

</div>

<div class="flex-parent has-child">

   <div class="flex-child long-and-truncated-with-child-corrected">
       <h2>3. This is a long string that is OK to truncate please and thank you</h2>
   </div>

   <div class="flex-child short-and-fixed">
       <div></div>
       <div></div>
       <div></div>
   </div>

</div>

.flex-parent {
  display: -webkit-box;
  display: flex;
  -webkit-box-align: center;
  align-items: center;
  padding: 10px;
  margin: 30px 0;
}
   
h2 {
  font-size: 1rem;
  font-weight: normal;
}
   
.long-and-truncated {
  -webkit-box-flex: 1;
  flex: 1;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
   
.short-and-fixed {
  white-space: nowrap;
}
   
.short-and-fixed>div {
  width: 30px;
  height: 30px;
  border-radius: 10px;
  background: lightgreen;
  display: inline-block;
}
   
.long-and-truncated-with-child {
  -webkit-box-flex: 1;
  flex: 1;
}
   
.long-and-truncated-with-child h2 {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
   
.long-and-truncated-with-child-corrected {
  -webkit-box-flex: 1;
  flex: 1;
  min-width: 0;
  /* or some value */
}
   
.long-and-truncated-with-child-corrected h2 {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
   
body {
  padding: 40px;
}
{% endcodeblock %}

Reference:  

* [Flexbox and Truncated Text](https://css-tricks.com/flexbox-truncated-text/)  


