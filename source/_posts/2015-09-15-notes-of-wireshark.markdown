---
layout: post
title: "Notes of Wireshark"
date: 2015-09-15 09:05:51 +0800
comments: true
categories: [Archives]
keywords: Wireshark, Capture package
discription: Notes of Using Wireshark
---

###1.Capture HTTP traffic on your server

```
Capture > Capture Filters... > New > OK
Capture > Options... > Capture Filter(Set to appropriate filter) > Start

host www.baidu.com

curl -d "name=test&page=1" http://www.baidu.com
```
```
POST / HTTP/1.1
User-Agent: curl/7.37.1
Host: www.baidu.com
Accept: */*
Content-Length: 16
Content-Type: application/x-www-form-urlencoded

name=test&page=1


HTTP/1.1 302 Moved Temporarily
Date: Tue, 15 Sep 2015 00:56:03 GMT
Content-Type: text/html
Content-Length: 215
Connection: Keep-Alive
Location: http://www.baidu.com/search/error.html
Server: BWS/1.1
X-UA-Compatible: IE=Edge,chrome=1
BDPAGETYPE: 3
Set-Cookie: BDSVRTM=0; path=/

<html>
<head><title>302 Found</title></head>
<body bgcolor="white">
<center><h1>302 Found</h1></center>
<hr><center>pr-nginx_1-0-245_BRANCH Branch
Time : Thu Sep 10 11:36:26 CST 2015</center>
</body>
</html>
```

Reference:https://wiki.wireshark.org/CaptureFilters

