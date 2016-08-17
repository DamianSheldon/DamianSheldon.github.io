---
layout: post
title: "网络是如何工作的"
date: 2014-08-26 21:20:31 +0800
comments: true
published: true
categories: [Archives] 
keywords: Internet, TCP/IP, HTTP
description: How does the Internet work
---

我一直想弄清楚互联网上两台计算机是如何通信的？

现实生活中，我们经常会和其他人交谈，交谈的双方得选择一门两个人都懂的语言，如果一个人说英语，另一个人不懂英语，那么对话就没办法继续下去；如果两人置身于真空中（暂不考虑其他情况），直接对话也不可能，没有了传输的介质。类比于人类的交谈，我们可以简单的推断计算机之间要实现通信需要一些充分条件：1.两台计算机；2.使用共同理解的语言；3.有介质传输信息。

找两台计算机比较好办，让计算机使用共同的语言就棘手些。计算机本身只能处理二进制数，如果想要表达我们想说的一句话，很自然需要很多位二进制来表示，这个之间的映射关系是字符编码，如今通用的是 Unicode 编码。我们说话是一句一句的说，计算机之间通信也差不多，只不过我们把它称为包。那一个包就是包含一句话啰！计算机在这方面并不比人类，一个包有时可以包含一句话，有时可能不行，它是固定长度的。计算机前辈规定一个包的最长长度是1500个字节。它们之前的传输介质有很多，可以分为两大类：有线；无线。虽然形式各样，但它们的主要任务就是充当传输介质，让一台计算机可以把0，1发到另一台计算机。而这个发送0，1的装置我们称为网卡。

现在两台计算机可以收发数据了，但离我们的目标还有一段距离，我们想要的是任意两台计算机之间的通信。每张网卡都有一个唯一标识，我们称为 MAC(Media Access Control)，由于 MAC 地址是设计用来在机器之间的收发数据的，所以用它来进行网络间的通信并不合适。于是我们引入了 IP(Internet Protocol) 地址，它能唯一标识网络上的一台计算机。组建网络的时候我们需要把所有的计算机都连接起来，为了完成这个目标，我们引入了路由器(Router)。它的工作原理大致是这样的：它内部存有一张路由表，收到数据包之后，取出 IP 地址查看，然后决定转发给相应的路由器或者连接的机器。

人们就是这样组建出了庞大的互联网，它的结构可以用下图来示意：

![Internet_Connectivity_Distribution & Core](../images/550px-Internet_Connectivity_Distribution_\&_Core.svg.png)

![Internet_Connectivity_Access_layer](../images/440px-Internet_Connectivity_Access_layer.svg.png)

互联网的组网方式并不是一台计算机直接连接路由器，然后这个路由器通过若干个路由器和另外一台计算机相连。实际上由于联网的计算机非常之多，网络发展成了层级关系，最上层是称为 Tier 1 network， 下面还有 Tier 2 network 和 Tier 3 network。ISP(Internet Service Provider) 通常就是接入的 Tier 3 network 或 Tier 2 network，终端的用户则是通过 ISP 接入网络。

Reference:

1.[How Does the Internet Work?](https://web.stanford.edu/class/msande91si/www-spr04/readings/week1/InternetWhitepaper.htm)  
2.[History of the Internet](https://en.wikipedia.org/wiki/History_of_the_Internet)   
3.[Tier 1 network](https://en.wikipedia.org/wiki/Tier_1_network)  
4.[Tier 2 network](https://en.wikipedia.org/wiki/Tier_2_network)  
5.[Autonomous system](https://en.wikipedia.org/wiki/Autonomous_system_(Internet))  
6.[Internet service provider](https://en.wikipedia.org/wiki/Internet_service_provider)  
7.[Who provides the Internet service to Internet Service Providers (ISPs)?](http://superuser.com/questions/399300/who-provides-the-internet-service-to-internet-service-providers-isps)  
8.[Internet Routing and Traffic Engineering](https://www.awsarchitectureblog.com/2014/12/internet-routing.html)

