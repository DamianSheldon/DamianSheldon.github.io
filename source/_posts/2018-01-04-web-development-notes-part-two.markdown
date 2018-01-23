---
layout: post
title: "Web 开发问题汇总(二)"
date: 2018-01-04 10:08:36 +0800
comments: true
categories: [Archives, Web Development]
keywords: 
description: Web Development Notes Part Two
---

###1.Uncaught TypeError: Cannot read property 'msie' of undefined
A:The $.browser method has been removed as of jQuery 1.9.

```
>jQuery.browser() removed

>The jQuery.browser() method has been deprecated since jQuery 1.3 and is removed in 1.9. If needed, it is available as part of the jQuery Migrate plugin. We recommend using feature detection with a library such as Modernizr.

>— jQuery Core 1.9 Upgrade Guide.
```

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


