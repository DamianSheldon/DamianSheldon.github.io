---
layout: post
title: "iOS App 开发问题汇总（六）"
date: 2016-03-28 20:14:53 +0800
comments: true
categories: [Archives, iOS Development]
keywords: iOS
discription: iOS development problems
---
### 1. The data couldn’t be read because it isn’t in the correct format.
A:项目是2014年开发的，打包导出 Ad-hoc 时报上面的错误，Build Setting > Enable Bitcode > NO 之后再试成功了。

Reference:http://stackoverflow.com/questions/33995769/ipatool-fails-to-build-with-bitcode-xcode-7-1-1

### 2.setNeedsLayout vs. setNeedsUpdateConstraints and layoutIfNeeded vs updateConstraintsIfNeeded
A:

* If you manipulated constraints directly, call setNeedsLayout.
* If you changed some conditions (like offsets or smth) which would change constraints in your overridden updateConstraints method (a recommended way to change constraints, btw), call setNeedsUpdateConstraints, and most of the time, setNeedsLayout after that.
* If you need any of the actions above to have immediate effect—e.g. when your need to learn new frame height after a layout pass—append it with a layoutIfNeeded.

Reference:http://stackoverflow.com/questions/20609206/setneedslayout-vs-setneedsupdateconstraints-and-layoutifneeded-vs-updateconstra


