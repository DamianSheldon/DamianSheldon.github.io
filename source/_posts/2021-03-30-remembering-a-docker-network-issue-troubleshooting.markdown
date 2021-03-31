---
layout: post
title: "记一次 docker 网络问题排查"
date: 2021-03-30 14:40:26 +0800
comments: true
categories: [Archives, Web Development]
keywords: docker, iptables, firewall
description: Remembering a docker network issue troubleshooting
---
最近在学 docker，虽然很早之前就简单体验过一次，但限于时间没有深入，最近有点空余时间，于是准备深入学习一下。但没想到一开始就遇到了拦路虎，照着官方文档 Get Started 一步一步往下，在构建一个自己的镜像时就遇到了问题，在多容器之间也无法通信，这可愁坏我了，只能硬着头皮来排查问题。

构建镜像时问题表现是牵涉到连接网络的命令会失败，google 一圈之后，找到使用 `--network=host` 的偏方，成功绕过问题。而多容器之间无法通信却一时措手无策。现在就只有 `--network=host` 这一个线索，于是就想加上这个参数后有什么区别呢？  

在官方文档上搜寻一圈之后我没查到有用信息，于是我想相关的书可能会讲讲 docker 网络这块，在微信读书上找了杨保华、戴王剑和曹亚仑合著的《Docker 技术入门与实战(第3版)》。因为 docker 技术在快速发展，所以书中的命令与当前版本有些许差异，但问题不大，不妨碍理解。这本书有两节专门介绍 docker 网络，于是我便先看了这两节，从这里得到一条重要线索：docker 和宿主机的通信是依靠防火墙转发。`--network=host` 也验证了这条线索，加上这参数会直接使用宿主机的网络配置，这样 docker 和宿主机的通信就不需要防火墙转发了。  

问题现在定位到了防火墙，那么为什么防火墙不转发 docker 的网络数据包呢？是不是哪条防火墙规则没配对？另外我还怀疑是不是我的实践环境有问题？我的实践环境是这样的:物理主机是 macOS，上面安装 virtulbox，使用 virtulbox 创建了一个 CentOS 8 的虚拟机, 虚拟机的网络模式是 NAT。这种情况无疑增加了问题排查的难度，怎么来排查呢？  

我先看了一下 virtulbox 的用户手册中的网络模式介绍，NAT 是可以连接到宿主机，而且现在虚拟机是可以访问网络的。如果想快速定位问题，我想还得从网络请求的数据包入手，想办法来跟踪数据包。  

于是就来查怎么调试 iptables。查到可以通过 TRACE 和 LOG 来输出日志，看介绍也没看出这俩有什么区别。凭经验觉得 TRACE 好像比 LOG 后出来，看起来也高大上一点，先试着用 TRACE 。Google 了一圈，找到都是 CentOS 6 或 CentOS 7 相关的配置，也只能先将就着用吧。  

我先是参考的 [CentOS implements iptables log output and debugging through raw table](https://www.programmersought.com/article/68601428960/)  


```
$sudo modprobe ipt_LOG
$sudo sysctl net.netfilter.nf_log.2

```

输出的结果是 NONE，于是尝试显示设置 `sudo sysctl net.netfilter.nf_log.2=ipt_LOG`
<!--more-->
结果给我报一个 `sysctl: setting key "net.netfilter.nf_log.2": No such file or directory`, 为什么没有这个 key 呢？于是我查询一下:  

```
$ sudo sysctl -a | grep net.netfilter.nf_log
net.netfilter.nf_log.0 = NONE
net.netfilter.nf_log.1 = NONE
net.netfilter.nf_log.10 = NONE
net.netfilter.nf_log.11 = NONE
net.netfilter.nf_log.12 = NONE
net.netfilter.nf_log.2 = NONE
net.netfilter.nf_log.3 = NONE
net.netfilter.nf_log.4 = NONE
net.netfilter.nf_log.5 = NONE
net.netfilter.nf_log.6 = NONE
net.netfilter.nf_log.7 = NONE
net.netfilter.nf_log.8 = NONE
net.netfilter.nf_log.9 = NONE
net.netfilter.nf_log_all_netns = 0
```

明明有这个 key 啊, Google 一圈不得要领。继续搜索, 找到这篇 [Tracing iptables on CentOS – Cheat sheet](https://vocon-it.com/2020/03/30/tracing-iptables-on-centos-cheat-sheet/), 它提到的方法如下:  

```
modprobe nf_log_ipv4
sudo sysctl net.netfilter.nf_log.2=nf_log_ipv4
```

因为我对 linux 模块相关命令还是有点了解，知道 `modprobe` 是用来加载模块， `modinfo` 可以查看模块的信息,我发现 `nf_log_ipv4` 这个模块在我的系统中已经加载，所以我就直接尝试 `sudo sysctl net.netfilter.nf_log.2=nf_log_ipv4`, 得到的仍然是 `sysctl: setting key "net.netfilter.nf_log.2": No such file or directory`，不得不说这很迷，为什么会一直报这么一个不相关的错误。  

没办法，继续搜索, 找到 [IPTables. Setting nf_log kernel parameter](https://forums.centos.org/viewtopic.php?t=54411),它其中提到：  

>I think they split the xt_LOG code in newer kernel versions and you need to modprobe nf_log_ipv4 now and sysctl net.netfilter.nf_log.2=nf_log_ipv4 (assuming you want to trace ipv4 packets)  

很奇怪，为什么其他人都可以设置，我这里却不行，而且这三篇文章用的方法都类似，不至于啊，google 到的也就这几篇文章，于是我回过头去再次研读第一篇文章，在想要不要试下作者提供 CentOS 7 系列的配置方法，毕竟版本更接近，而且这三篇文章都提到用 `nf_log_ipv4`, 这时我发现他的方法里 `modprobe nf_log_ipv4` 之后并不需要设置， `net.netfilter.nf_log.2` 的值便设置了,于是我也照做，终于配置成功。我觉得这里可能是 CentOS 8 有 bug，因为明明有 key 却设置不上，现在设置成功，也就不管那么多了，继续解决问题要紧。  

我到 `/etc/rsyslog.conf` 中开启内核日志输出：  

```
kern.*                                                 /var/log/messages
```

重启日志输出服务 `sudo systemctl restart rsyslog.service`,往防火墙里添加 TRACE 规则:  

```
sudo iptables -t raw -A OUTPUT     -p tcp -j TRACE
sudo iptables -t raw -A PREROUTING -p tcp -j TRACE
```

监视系统日志 `sudo tail -f /var/log/messages`

重启 docker 服务 `sudo systemctl restart docker.service`, 运行容器 `sudo docker run -it busybox`, 然后在 busybox 容器中触发网络请求:  

```
cd /tmp/
wget http://www.bing.com/
```

系统日志并没有输出， 查看 `sudo dmesg` 也没有日志输出。真是让挫败啊，没办法，只能退而求其次改用 LOG。  


```
# 删除 TRACE 规则
sudo iptable -t raw -D PREROUTING 1
sudo iptable -t raw -D OUTPUT 1

## 添加 LOG 规则
sudo iptables -t raw -A OUTPUT     -p tcp -j LOG --log-level debug
sudo iptables -t raw -A PREROUTING     -p tcp -j LOG --log-level debug
```

同样重启日志服务 `sudo systemctl restart rsyslog.service`, 监视系统日志 `sudo tail -f /var/log/messages`，重启 docker 服务 `sudo systemctl restart docker.service`, 运行容器 `sudo docker run -it busybox`, 然后在 busybox 容器中触发网络请求:  

```
cd /tmp/
wget http://www.bing.com/
```

日志是成功输出了，内心着实高兴了一把，但由于我的规则设置太宽泛，输出的太多了，很难找到有用的信息，脑中闪过一个念头，那把规则设置更严格一点不就可以了，搓搓小手，兴奋地实践起来，于是我把规则调整成容器发出的网络数据包：  

```
sudo iptables -t raw -A OUTPUT     -p tcp -s 172.17.0.0/16 -j LOG --log-level debug
sudo iptables -t raw -A PREROUTING     -p tcp -s 172.17.0.0/16 -j LOG --log-level debug
```

果然，相关日志输出少多了，只有两条,具体如下: 

```
Mar 29 15:51:46 centos kernel: IN=docker0 OUT= PHYSIN=vethfeab609 MAC=02:42:5d:dd:2d:af:02:42:ac:11:00:02:08:00 SRC=172.17.0.2 DST=222.246.1
29.80 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=30005 DF PROTO=TCP SPT=60366 DPT=80 WINDOW=29200 RES=0x00 SYN URGP=0 
Mar 29 15:51:47 centos kernel: IN=docker0 OUT= PHYSIN=vethfeab609 MAC=02:42:5d:dd:2d:af:02:42:ac:11:00:02:08:00 SRC=172.17.0.2 DST=222.246.1
29.80 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=30006 DF PROTO=TCP SPT=60366 DPT=80 WINDOW=29200 RES=0x00 SYN URGP=0 
```

这对解决问题帮助不大，摔！这几乎要击垮我了，但还是心有不甘。于是只能把希望再次寄托给 TRACE，我在 CentOS 8 上直接 `man iptables`，发现它和网络上找到的 man page 内容确实有些差异, 它的 target 专门独立到 `iptables-extensions`:

>iptables  can  use  extended packet matching and target modules.  A list of these is available in the iptables-extensions(8) man‐page.

继续 `man 8 iptables-extensions`，搜索 TRACE, 相关介绍如下:

> This  target  marks  packets so that the kernel will log every rule which match the packets as those traverse the tables, chains,rules. It can only be used in the raw table.

> With iptables-legacy, a logging backend, such as ip(6)t_LOG or nfnetlink_log, must be loaded for this to be visible.  The packets are  logged  with the string prefix: "TRACE: tablename:chainname:type:rulenum " where type can be "rule" for plain rule, "return" for implicit rule at the end of a user defined chain and "policy" for the policy of the built in chains.

> With iptables-nft, the target is translated into nftables' meta nftrace expression. Hence  the  kernel  sends  trace  events  via netlink to userspace where they may be displayed using xtables-monitor --trace command. For details, refer to xtables-monitor(8).

原来配合 `iptables-nft` 时, 日志是使用  `xtables-monitor --trace`, 似乎又看到了一丝曙光,于是赶紧删除 LOG 规则，添加 TRACE 规则:  

```
# 删除 LOG 规则
sudo iptable -t raw -D PREROUTING 1
sudo iptable -t raw -D OUTPUT 1

# 添加 TRACE 规则
sudo iptables -t raw -A OUTPUT     -p tcp -j TRACE
sudo iptables -t raw -A PREROUTING -p tcp -j TRACE
```

准备就绪之后就按前面相关的步骤触发网络请求，果然成功输出日志，而且输出了很多日志，我往屏幕下面翻，最后竟然输出的 "Failed to received netlink message: No buffer space available",为什么报错了呢？我尝试 `man xtables-monitor`,文档上明明说的是： xtables-monitor will run until the user aborts execution, typically by using CTRL-C.我这里为什么报错终止了呢？Google 一圈一无所获，想到之前 LOG 因为规则太宽松输出了很多日志，就想是不是这规则太宽松导致日志太多，缓冲区不够用，赶紧调整 TRACE 规则:  

```
# 删除 TRACE 规则
sudo iptable -t raw -D PREROUTING 1
sudo iptable -t raw -D OUTPUT 1

# 添加 TRACE 规则
sudo iptables -t raw -A OUTPUT     -p tcp -s 172.17.0.0/16  -j TRACE
sudo iptables -t raw -A PREROUTING -p tcp -s 172.17.0.0/16 -j TRACE
```

这下好了，日志正常输出，也没有报错了。于是仔细查看日志，终于找到： `firewalld:filter_FORWARD:rule:0x93:DROP`,原来是 firewall 将包丢掉了。可是对 firewall 一点也不熟，先找了一遍 firewall 教程看了一下，但是还不知道怎么在 firewall 中添加规则，要学会添加规则也得花点力气，这时脑海中冒出另一个想法：我何不直接停止 firewall，确认下是不是 firewall 导致的问题。

```
sudo systemctl stop firewalld.service
sudo systemctl stop docker.service
sudo systemctl start docker.service
sudo docker run -it busybox
```

尝试一下,发现可以了，确定确实是 firewall 导致的问题。那么现在是自己研究 firewall 添加规则吗？有点复杂啊，我在想是不是我 docker 版本太低了，检查一下确实比最新的版本要低，新版本可能解决了。在文档也确实找到了对应描述:  

>If you are running Docker version 20.10.0 or higher with firewalld on your system with --iptables enabled, Docker automatically creates a firewalld zone called docker and inserts all the network interfaces it creates (for example, docker0) into the docker zone to allow seamless networking.

于是升级新版本，新版本也确实解决了这个问题，嗯，这下暂时不用研究 firewall 添加规则了。

这次能成功排查 docker 的网络问题还是挺开心的，说实话一开始我是没信心的，毕竟才开始接触 docker，另一方面 linux 网络这块牵涉到的知识非常多，甚至在心里想实在解决不了就算了,直接在 macOS 上来学好了。得到的启发是不管有没有解决问题，首先解决问题的方法要对，方法对了之后还得要坚持；其次是注意自己软件使用的版本，相关的命令参数可能需要查看对应版本的文档去核实；最后是虽然项目的官方文档是入门的不错材料，但很多时候官方文档写得很浅或者对新手不友好，这时候如果觉得项目值得投入时间学习的话，看相关的书籍是一个很好的选择，对系统学习和精进大有禆益。

#Reference

* [CentOS implements iptables log output and debugging through raw table](https://www.programmersought.com/article/68601428960/) 
* [Tracing iptables on CentOS – Cheat sheet](https://vocon-it.com/2020/03/30/tracing-iptables-on-centos-cheat-sheet/)  
* [IPTables. Setting nf_log kernel parameter](https://forums.centos.org/viewtopic.php?t=54411)  
* [What is the relation between docker0 and eth0?](https://stackoverflow.com/questions/37536687/what-is-the-relation-between-docker0-and-eth0)  
* [Debugging rules in Iptables](https://serverfault.com/questions/78240/debugging-rules-in-iptables)  
* [How to configure Centos 7 firewallD to allow docker containers free access to the host's network ports?](https://unix.stackexchange.com/questions/199966/how-to-configure-centos-7-firewalld-to-allow-docker-containers-free-access-to-th)  
* [How To Set Up a Firewall Using firewalld on CentOS 8](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-8)  
* [Enable and Disable firewalld](https://firewalld.org/documentation/howto/enable-and-disable-firewalld.html)  
* 


