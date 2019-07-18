---
layout: post
title: "记一次换新 iPhone"
date: 2018-03-17 10:20:21 +0800
comments: true
categories: [Archives]
keywords: backup, corrupt, incompatible, iPhone, 换机
description: 一次换新 iPhone 的经过，过程中遇到了从 iTunes 备份中恢复遇到 your backup is corrupt or incompatible 的问题，以及我的解决办法。
---

说来惭愧，一直还是在用公司配的开发机 5S, 没用上最新手机的主要原因可以归结为穷 ><。但最近 5S 的存储不够用了，再加上心仪的 8 降价了，一冲动就入了一个，嗯，这几个月做好了吃土的准备了。  

iPhone 可以将旧手机数据迁移到新手机，这是我喜欢它的一个地方。苹果这篇[将内容从旧 iOS 设备传输到新 iPhone、iPad 或 iPod touch](https://support.apple.com/zh-cn/HT201269)详细介绍几种可行方法，我们可以从中选择一种方法。其实我觉得苹果应该出篇换机指南的，上面这篇文章的题目不是很容易让人把它与换机联系起来，也不是很容易记，这是我想记这么一笔的原因之一，N 年之后我再换 iPhone 的时候也可以按这篇文章的思想来指导自己迁移数据。

这里我选择使用 iTunes 备份恢复，快速开始功能好像不是很灵光，也可能是我不会用，而 iCloud 备份是通过网络，网速会影响备份和恢复，加上最近 iPhone 的 bug 贼多，这让我不是很放心。在此之前有使用过 iTunes 备份恢复，想着即使恢复过程出了问题备份还在，所以最终选择 iTunes。

备份过程很顺利，恢复过程就曲折了，iTunes 提示 backup is corrupt or incompatible，墨菲定理生效了，另一方面也佐证了选择 iTunes 是正确的选择，苹果什么时候能把这软件质量抓一抓，别只顾着赚钱啊。搜索找到了苹果对这个问题的解决办法：  
<!--more-->
> If an alert says that your backup is corrupt or incompatible  
> 
> If iTunes can't restore from a backup because the backup is corrupt or incompatible, make sure that iTunes is updated. If you see an error that says your iOS software is too old, find out how to update your device to restore the backup. If you still can't restore the backup, you might not be able to use that backup. Try to use an alternate backup or an iCloud backup, or Contact Apple Support for more help.

检查 iTunes 的版本，显示是最新的，这是什么幺蛾子,难道就这么放弃，不存在的，我有那么容易放弃吗，开玩笑。既然有所谓的版本的问题，我怀疑是 8 的版本和 5S 的版本不一致，于是两手准备：一方面对 5S 做一次全量 iCloud 备份，如果稍后 iOS 版本都一致了 iTunes 备份还不行就使用 iCloud 备份恢复；另一方面把 8 和 5S 升级到最新的 iOS，8 用 USB 接上 iTunes 是可以升级的。8 和 5S 都升级成最新版本的系统后，又对 5S 做了一次 iTunes 备份，之后 8 用这个备份顺利恢复了。  

回顾整个恢复过程感觉还挺曲折，真心希望苹果抓抓软件质量，别让 iOS 成了 bugOS。过程中 8 和 5S 只是小版本不一致却导致恢复不成功，还挺伤害用户体验的，毕竟好几年才使用一次的杀手锏功能之一，当我要使用的时候你却告诉我: "对不起，这次用不了!"我用新 iPhone 的喜悦感都被抛到九霄云外了。

### Reference  

* [将内容从旧 iOS 设备传输到新 iPhone、iPad 或 iPod touch](https://support.apple.com/zh-cn/HT201269)  
* [从数据迁移到处理旧 iPhone，这份换机指南都帮你想好了](https://sspai.com/post/41612)  
* [If your iTunes backup couldn‘t be completed or you can‘t restore from a backup](https://support.apple.com/en-us/HT203271)  
* [出售、赠送或折抵换购 iPhone、iPad 或 iPod touch 前该怎么做](https://support.apple.com/zh-cn/HT201351)  




