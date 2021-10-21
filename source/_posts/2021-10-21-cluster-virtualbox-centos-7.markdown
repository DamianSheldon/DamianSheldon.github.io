---
layout: post
title: "VirtualBox 搭建 CentOS 7 集群"
date: 2021-10-21 15:57:18 +0800
comments: true
categories: [archives]
keywords: Cluster, VirtualBox, CentOS7, Bridged networking, Internal networking 
description: Cluster VirtualBox CentOS 7
---

软件开发时我们可能需要集群环境，特别是现在微服务很流行的情况下。如果我们手里的电脑配置不错就可以用 来搭建集群，用于学习或开发还是很不错，性价比很高。  

我这里是用 VirtualBox 搭建 CentOS 7 集群，宿主机是内存 16 GB 加 SSD 1 TB 的 MacBook Pro。集群的核心是选择网络模式，VirtualBox 的网络模式概况如下:  

| Mode | VM -> Host | VM <- Host | VM1 <-> VM2 | VM -> Net/LAN | VM <- Net/LAN |
| --- | --- | --- | --- | --- | --- |
| Host-only | + | + | + | - | - |
| Internal | - | - | + | - | - |
| Bridged | + | + | + | + | + |
| NAT | + | Port forward | - | + | Port forward |
| NAT service | + | Port forward | + | + | Port forward |

我希望集群的机器可以互相访问并能访问网络，从上面的列表可知，我们可以选择 Bridged，或者 NAT service 搭配 Port forward。在搭建之初，我搜索了相关的资料，但是并没找到特别理想的，这篇 [Setting up CentOS 7 nodes](https://subscription.packtpub.com/book/web-development/9781785288685/1/ch01lvl1sec09/setting-up-centos-7-nodes) 勉强还凑合，于是我主要参考它来搭建，网络模式也同样是选择的 Bridged，它还使用了双网卡，虚拟机之间通信使用的 Internal Networking， 感觉上性能可能会好点，没实测。双网卡并不是必须的，VirtualBox 对 Internal Networking 的说明是其在安全方面更有优势:  

> Even though technically, everything that can be done using internal networking can also be done using bridged networking, there are security advantages with internal networking. In bridged networking mode, all traffic goes through a physical interface of the host system. It is therefore possible to attach a packet sniffer such as Wireshark to the host interface and log all traffic that goes over it. If, for any reason, you prefer two or more VMs on the same machine to communicate privately, hiding their data from both the host system and the user, bridged networking therefore is not an option.

由于 CentOS 主要是运行服务，我们可以使用 Minimal 安装，这样可以减少资源开销，安装好后，默认是没有启用网络的， 网络相关的配置是在 `/etc/sysconfig/network-scripts` 目录下，配置文件的命令惯例是 `ifcfg-enp0sX` ， `X` 是整数，我这里是 `ifcfg-enp0s3` 和 `ifcfg-enp0s8`，将 `/etc/sysconfig/network-scripts/ifcfg-enp0s3` 和 `/etc/sysconfig/network-scripts/ifcfg-enp0s8` 中的 `ONBOOT` 改成 `yes`。  

构建 Internal Network 时，手动去指定 ip 会很麻烦，VirtualBox 给了我们另一个选择:  

> Unless you configure the virtual network cards in the guest operating systems that are partici- pating in the internal network to use static IP addresses, you may want to use the DHCP server that is built into Oracle VM VirtualBox to manage IP addresses for the internal network.  

我们可以 `10.0.0.0/8`, `172.16.0.0/12` 和 `192.168.0.0/16` 选择一个合适的私有 ip 地址范围来构建 Internal Network，我这里选择 `172.16.0.0/12`。  

在宿主机下运行命令 `/Applications/VirtualBox.app/Contents/MacOS/VBoxManage dhcpserver add --netname=intnet --server-ip=172.16.0.1 --netmask=255.240.0.0 --lower-ip=172.16.0.2 --upper-ip=172.16.255.255 --enable` 创建好 DHCP server。

之后我们可以利用 VirtualBox 的克隆功能来扩展我们的集群节点。  

#修改记录
	
* 2021/10/21：第一次完成

#Reference  

* [Setting up CentOS 7 nodes](https://subscription.packtpub.com/book/web-development/9781785288685/1/ch01lvl1sec09/setting-up-centos-7-nodes)  
* [Building an Internal Network in VirtualBox](https://54m4ri74n.medium.com/building-an-internal-network-in-virtualbox-d0a4974882d0)  



