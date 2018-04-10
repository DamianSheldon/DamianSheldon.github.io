---
layout: post
title: "Web 开发问题汇总(二)"
date: 2018-01-04 10:08:36 +0800
comments: true
categories: [Archives, Web Development]
keywords: web, jquery, servlet, eclipse
description: Noting problems encounter during web development, every fifteen problem produce a blog, this is the second.
---

###1.Uncaught TypeError: Cannot read property 'msie' of undefined
A:The $.browser method has been removed as of jQuery 1.9.


> jQuery.browser() removed

> The jQuery.browser() method has been deprecated since jQuery 1.3 and is removed in 1.9. If needed, it is available as part of the jQuery Migrate plugin. We recommend using feature detection with a library such as Modernizr.

> — jQuery Core 1.9 Upgrade Guide.


As stated in the Upgrade Guide you can try using the [jQuery Migrate plugin](https://github.com/jquery/jquery-migrate/) to restore this functionality and let jQuery Tools work.

Reference:[Uncaught TypeError: Cannot read property 'msie' of undefined - jQuery tools](http://stackoverflow.com/questions/14923301/uncaught-typeerror-cannot-read-property-msie-of-undefined-jquery-tools)  

###2.The superclass “javax.servlet.http.HttpServlet” was not found on the Java Build Path in Eclipse
A:To include http-servlet into your class path, you have two options:  

	1. In this solution, you can add desired server runtime into your application as project facet. As runtime servers have already servlet runtime dependencies, they get included into your project and hence error is gone.
	2. Another option is include the servlet dependency through maven itself. This will also fix the error.
	
```
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
</dependency>
```

Reference:[The superclass “javax.servlet.http.HttpServlet” was not found on the Java Build Path in Eclipse](https://howtodoinjava.com/tools/eclipse/solved-the-superclass-javax-servlet-http-httpservlet-was-not-found-on-the-java-build-path-in-eclipse/)  

###3.Dynamic Web Module 3.1 requires Java 1.7 or newer
A:

Step1:  
Make sure your Java Project is configured probably to use Java 1.7.
Right click your project > Properties > Java Compiler and set “Compiler compliance level” to 1.7

Step2:  
Java Build Path. Click “Edit“. And change it to “Java 7”

Step3:  
Next from the menu on the left select Project Facets > Java and set its version to 1.7

Extra Tips:  
If you have maven project try updating source and target version of java compiler to 1.7 in pom.xml file.

Reference:[How to solve “Dynamic Web Module 3.1 requires Java 1.7 or newer” in Eclipse](https://crunchify.com/how-to-solve-dynamic-web-module-3-1-requires-java-1-7-or-newer-in-eclipse/)  

###4.How to display available goals of maven plugin?
A:

> Recent Maven plugins have generally an help goal to have in the command line the description of the plugin, with their parameters and types. For instance, to understand the javadoc goal, you need to call:

```bash
$ mvn javadoc:help -Ddetail -Dgoal=javadoc
```

It works of course for Maven core plugins. to list all goals of the archetype plugin :  

```bash
$ mvn archetype:help
```

And it works also for third party plugins. For example, to display all goals of the spring-boot plugin :  

```bash
$ mvn spring-boot:help
```

Reference:[How to display a list of available goals?](https://stackoverflow.com/questions/1674524/how-to-display-a-list-of-available-goals)  

###5.Not a java source folder
A:以 maven-archetype-webapp 为原型新建工程，建好的工程没有 java 目录，于是在 src/main 下新建 java 目录，之后新建 Filter 或 Servlet 时提示 Not a java source folder.尝试右击 java 目录 > Build Path > Use as Source Folder 依然未生效，只得将目录删除恢复原状想其他办法。  

于是选中工程 > Properties > Java Build Path > Source， 里面提示 src/main/java build path entries are missing, 嗯，有了新的线索，于是将这些丢失的入口都删除了，再重新创建 java source folder, 这样问题得到了解决，感觉这个问题像是 eclipse 的一个 bug.
<!--more-->
###6.Thymeleaf 如何支持遍历枚举
A: 

> The java.util.List class isn’t the onlyvalue that can be used for iteration in Thymeleaf. There is a quite complete set of objects that are considered iterable by a th:each attribute:  

> * Any object implementing java.util.Iterable
> * Any object implementing java.util.Enumeration .
> * Any object implementing java.util.Iterator , whose values will be used as they are returned by the iterator, without the need to cache all values in memory.
> * Any object implementing java.util.Map . When iterating maps, iter variables will be of class java.util.Map.Entry . Any array.
> * Any other object will be treated as if it were a single-valued list containing the object itself.

上面就是可选方案，一开始我没看仔细，以为 Enumeration 可以直接遍历，之后细看是实现 java.util.Enumeration 的对象，这样综合来看将 Enumeration 转化成 List 是不错的选择。  

```html
<div th:each="attr: ${java.util.Collections.list(#httpServletRequest.attributeNames)}">
```

Reference:[please allow to iterate java.util.Enumeration](https://github.com/thymeleaf/thymeleaf/issues/321)  

###7.Call method with iter vaiable as argument in thymeleaf  
A:

```html
<ul th:each="name : ${names}">
	<li
		th:text="|${name} = ${#session.getAttribute(
		name )}|">uid
		= 502</li>
</ul>
```

Reference:[invoking a method on model with parameter](http://forum.thymeleaf.org/invoking-a-method-on-model-with-parameter-td4026846.html)  

###8.Hiding text using text-indent  
A:

```html
<style type="text/css">

h1 a {
    text-decoration: none;
    position: absolute; /* Depending on your placement */
    width: 260px;
    height: 100px;
    bottom: 0px; /* Depending on your placement */
    background: url(images/aext-logo.png) no-repeat left top;
    text-indent: -99999px;
}

</style>

<h1><a href="http://aext.net">AEXT.NET</a></h1>
```

在尝试过程中发现要设置 width 和 height 才能生效，可以注意下。  

Reference:[CSS text-indent: An Excellent Trick To Style Your HTML Form](http://bloggingexperiment.com/css-text-indent-style-your-html-form)  

###9.SEVERE: Exception starting filter
A:详细错误信息如下：  

```
Apr 08, 2018 10:28:48 AM org.apache.catalina.core.StandardContext filterStart
SEVERE: Exception starting filter [DispatchFilter]
java.lang.ClassNotFoundException: com.tenneshop.bookmark.web.filter.DispatchFilter
	at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1291)
	at org.apache.catalina.loader.WebappClassLoaderBase.loadClass(WebappClassLoaderBase.java:1119)
	at org.apache.catalina.core.DefaultInstanceManager.loadClass(DefaultInstanceManager.java:544)
	at org.apache.catalina.core.DefaultInstanceManager.loadClassMaybePrivileged(DefaultInstanceManager.java:525)
	at org.apache.catalina.core.DefaultInstanceManager.newInstance(DefaultInstanceManager.java:150)
	at org.apache.catalina.core.ApplicationFilterConfig.getFilter(ApplicationFilterConfig.java:264)
	at org.apache.catalina.core.ApplicationFilterConfig.<init>(ApplicationFilterConfig.java:108)
	at org.apache.catalina.core.StandardContext.filterStart(StandardContext.java:4590)
	at org.apache.catalina.core.StandardContext.startInternal(StandardContext.java:5233)
	at org.apache.catalina.util.LifecycleBase.start(LifecycleBase.java:150)
	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1419)
	at org.apache.catalina.core.ContainerBase$StartChild.call(ContainerBase.java:1409)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
	at java.lang.Thread.run(Thread.java:745)

Apr 08, 2018 10:28:48 AM org.apache.catalina.core.StandardContext startInternal
SEVERE: One or more Filters failed to start. Full details will be found in the appropriate container log file
Apr 08, 2018 10:28:48 AM org.apache.catalina.core.StandardContext startInternal
SEVERE: Context [/bookmark] startup failed due to previous errors
```

从错误信息来看问题的原因是类找不到，于是可以到部署的目标路径下确认 `com.tenneshop.bookmark.web.filter.DispatchFilter`是否存在。从 eclipse 输出的 debug 日志中找到部署的目标路径：  

```
Apr 08, 2018 10:28:38 AM org.apache.catalina.startup.VersionLoggerListener log
INFO: Command line argument: -Dwtp.deploy=/Users/dongmeiliang/eclipse-workspace/.metadata/.plugins/org.eclipse.wst.server.core/tmp1/wtpwebapps
```

发现类确实不存在，重新 `$ mvn compile` 之后问题解决。回想下发觉问题产生的原因是我在终端 `$ mvn clean`，然后在 eclipse 中 refresh  project ，最后直接就 debug on server 导致没有包含编译的类。  

Reference:[Tomcat SEVERE: Exception starting filter CorsFilter, ClassNotFoundException](https://stackoverflow.com/questions/32692321/tomcat-severe-exception-starting-filter-corsfilter-classnotfoundexception)  

###10.如何改变 server context path?
A:用 mvn 新建工程时，artifactId 写成了 package name，之后在访问 web 时就略显尴尬，可以通过 Window > Show View > Servers > Double click specific server > Modules > Edit target web module。 

###11.Eclipse error: “The import XXX cannot be resolved”
A:With me it helped changing the compiler compliance level. For unknown reasons it was set to 1.6 and I changed it to 1.8.
Once at project level right click on project > Properties > Java Compiler, while in Eclipse click on menu Window > Preferences > Java > Compiler.

Reference:[Eclipse error: “The import XXX cannot be resolved”](https://stackoverflow.com/questions/4322893/eclipse-error-the-import-xxx-cannot-be-resolved)  

###12.Index downloads are disabled, search result may be incomplete.
A:

1.	In Eclipse, click **on Windows > Preferences**, and then choose **Maven** in the left side.
2.	Check the box "**Download repository index updates on startup**".
* Optionally, check the boxes **Download Artifact Sources** and **Download Artifact JavaDoc**.
3.	Click **OK**. The warning won't appear anymore.
4.	Restart Eclipse.

Reference:[How do I enable index downloads in Eclipse for Maven dependency search? ](https://stackoverflow.com/questions/24252256/how-do-i-enable-index-downloads-in-eclipse-for-maven-dependency-search)  

###13.How to Iterate Over a Map in Java
A:There are at least 4 method to iterate over a map, one is Iterating over entries using For-Each loop.  

```java
Map<Integer, Integer> map = new HashMap<Integer, Integer>();
for (Map.Entry<Integer, Integer> entry : map.entrySet()) {
    System.out.println("Key = " + entry.getKey() + ", Value = " + entry.getValue());
}
```

Reference:[How to Iterate Over a Map in Java](http://www.sergiy.ca/how-to-iterate-over-a-map-in-java/)  


