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

```
$ mvn javadoc:help -Ddetail -Dgoal=javadoc
```

It works of course for Maven core plugins. to list all goals of the archetype plugin :  

```bash
$mvn archetype:help
```

And it works also for third party plugins. For example, to display all goals of the spring-boot plugin :  

```
$ mvn spring-boot:help
```

Reference:[How to display a list of available goals?](https://stackoverflow.com/questions/1674524/how-to-display-a-list-of-available-goals)  

