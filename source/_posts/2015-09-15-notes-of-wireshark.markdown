---
layout: post
title: "Tcpdump 和 Wireshark 使用笔记"
date: 2015-09-15 09:05:51 +0800
comments: true
categories: [Archives]
keywords: tcpdump, Wireshark, Capture package
description: Notes of Using Wireshark and tcpdump.
---
Tcpdump 和 Wireshark 是两个常用的抓包工具，一个是命令行工具；一个图形界面工具，可以按自己的喜好选择。图形界面的 Wireshark 个人感觉使用更简单，但从打开到抓包花费的时间要比使用 Tcpdump 要多，有的场景也不能使用 Wireshark，例如在 Android 真机上抓包。由于使用它们时核心是一样的，就是按照包过滤语法(pcap-filter(7))来指定表达式，因此可以先使用 Wireshark 上手，之后熟练起来后在不能用 Wirehshark 的场景便可以使用 Tcpdump。

##Wireshark
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

##Tcpdump
###1.指定 Host 和 Port 抓包
在开发过程中，我们可能想查看客户端和服务器的详细交互，这时我们通过指定 Host 和 Port 抓包来得到详细的 HTTP 交互。例如现在是在我的本地开发环境，服务器的端口是8080，可以用以下命令:

```bash
$sudo tcpdump -i lo0 -vv '(src localhost and dst port 8080) or (dst localhost and src port 8080)'
```

##Reference: 

* [CaptureFilters](https://wiki.wireshark.org/CaptureFilters)  
* [A tcpdump Tutorial and Primer with Examples](https://danielmiessler.com/study/tcpdump/#src-dst-port)  
* [tcpdump 命令](https://www.ibm.com/support/knowledgecenter/zh/ssw_aix_71/com.ibm.aix.cmds5/tcpdump.htm)  


