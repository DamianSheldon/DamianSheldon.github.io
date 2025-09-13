---
layout: post
title:  "Arthas 排查函数调用异常"
date:   2025-02-04 17:40:38 +0800
categories: [archives]
---

最近生产环境有方法对于部分参数会调用异常，由于代码是很久之前的其他同事写的，当时并没有在最外层捕获异常并记录日志，所以并不知道什么原因，于是想利用 Arthas 来在线排查。

首先是想利用 watch 来观察方法的异常

```sh
watch com.github.damiansheldon.PatientInfoPrivateServiceImpl getPatientBaseInfoAndBalance "{params[0], throwExp}" -e -x 4
```

但只能看到抛出空指针异常，没有具体的堆栈信息，所以也没法定位具体是哪里有问题。

然后利用 `sc/jad/redefine` 一条龙来热更新，在方法最外层加上 try-catch, 并输出日志，但居然打印出堆栈也是空的。

以尝试用 tt，输出的堆栈还是空的。

再使用 trace, 这次终于定位是调到哪个方法报空指针了。

回顾总结一下，排查函数调用异常，先可以使用 watch，其次是 trace, 能快速定位到问题。

我愿称 Arthas 为 java 开发的神器，值得每个 java 开发拥有。