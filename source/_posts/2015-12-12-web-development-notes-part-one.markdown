---
layout: post
title: "Web Development Notes Part One"
date: 2015-12-12 22:27:22 +0800
comments: true
categories: [Archives, Web Development]
keywords: HTML, CSS, JavaScript, Bootstrap, ionic, GMP, AngularJS, JQuery
discription: Web Development Notes Part One
---

###1. What is href=“#” and why is it used?

Solution:A hashtag - # within a hyperlink specifies an html element id to which the window should be scrolled.  
href="#some-id" would scroll to an element on the current page such as <div id="some-id">.

href="//site.com/#some-id" would go to site.com and scroll to the id on that page.  

href="#" doesn't specify an id name, but does have a corresponding location - the top of the page. Clicking an anchor with href="#" will move the scroll position to the top.

Reference:http://stackoverflow.com/questions/4855168/what-is-href-and-why-is-it-used

###2. How to debug your web page, technology stack is LAMP.  

Solution:View log, figure out what cause problems.

```
$ tail -f /var/log/apache2/error_log
[Fri Dec 11 21:34:57.003657 2015] [:error] [pid 17465] [client ::1:50505] PHP Parse error:  parse error, expecting `','' or `';'' in /Users/dongmeiliang/Sites/knowledgeispower/includes/header.html on line 7
[Fri Dec 11 21:34:57.003701 2015] [:error] [pid 17465] [client ::1:50505] PHP Stack trace:
[Fri Dec 11 21:34:57.003723 2015] [:error] [pid 17465] [client ::1:50505] PHP   1. {main}() /Users/dongmeiliang/Sites/knowledgeispower/index.php:0
```
