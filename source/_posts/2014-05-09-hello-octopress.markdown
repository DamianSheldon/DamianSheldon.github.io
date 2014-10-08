---
layout: post
title: "Hello Octopress"
date: 2014-05-09 15:01:51 +0800
comments: true
categories: [Archives]
keywords: Octopress, blog, Problem, Solution
description: 用Octopress搭建自己博客过程中的常见问题及解决办法
---
Octopress:A blogging framework for hackers.

看到介绍就被吸引了，正好最近想写些东西，于是决定用Octopress来搭个自己的博客。Octopress既然是为黑客准备的，有点难度那也是很正常的。网上很多都是介绍搭建的博客的步骤，但对出现各种问题以及解决办法的总结不是很多。而自己和ruby不是好朋友，这过程中出现很多的问题，这里把遇到的问题贴上来，一来做个总结；二来也可以给遇到同样的问题的朋友一些帮助。

#问题1  
```bash
	An error occurred while installing RedCloth (4.2.9), and Bundler cannot continue.  
	Make sure that gem install RedCloth -v '4.2.9' succeeds before bundling.  
```

Solution:这个问题不知道怎么回事，网上找了很久也没有找到解决办法。看到Octopress需要ruby1.9.3以上，而我系统上的ruby是一个通用的版本，我试着用rbenv装了个2.1.1版本，并设置成全局版本，问题解决了，不过还是没有弄明白是什么原因。

#问题2  
```bash
	rake aborted!  
	You have already activated rake 10.1.0, but your Gemfile requires rake 10.0.4. Prepending `bundle exec` to your command may solve this.
	/Users/dongmeiliang/Documents/octopress/Rakefile:2:in `<top (required)>'
	(See full trace by running task with --trace)
```

Solution:这个问题可以每次加上bundle exec,但是总有一种不舒坦的感觉，找了很久找到一点线索，最后用下面办法解决了:  
```bash
	brew update
	brew doctor
	cd root_path_octopress
	git pull octopress master
	rm Gemfile.lock
	vim Gemfile
	gem 'rake', '~> 10.1.0'//改成合适的版本
```

#问题3  
```bash
	dongmeiiangsmbp:octopress dongmeiliang$ rake generate
	...
	Build Failed
```

Solution:网上找了下，类似问题的原因可能是有空格之类的问题，用FileMerge比较了下发现是空格的原因。吐槽下这原因感觉有点难以接受啊。十分感谢原帖作者，忘记把链接复制下来了。

#问题4
之前安装过程中，出于测试的目的，写了篇草稿发上来了，现在想重新把文章编辑下，但是奇怪的是source分支中找不到原文件了，如果你知道原因请告诉我一声。

Solution:我的解决办法是在_deploy路径下同步master分支，解决合并冲突，然后把blog目录下相应的index.html文件删除了，然后：
```bash
	rake generate
	rake deploy
```
	
##问题5（2014.5.21）
怎么把文章放到navigation对应的分类组织中呢？

solution:我在navigation中加了一个iOS Devlopment的分类是这么做的： 
```bash 
rake new_page[categories/iOS_Development]  
vim source/_includes/custom/navigation.html
```
增加<li><a href="{{ root_url }}/blog/categories/ios-development">iOS Development</a></li>

然后在文章的头部：
categories: [iOS Development]

##问题6
怎么把侧边栏放到底部去呢？
	
##问题7  
```bash
	Error fetching https://ruby.taobao.org/:
    	too many connection resets (https://rubygems-china.oss.aliyuncs.com/specs.4.8.gz)
```

Solution:网上查找相关问题，没有找到好的解决办法。经过一番琢磨，觉得可能是我设置了goagent，于是注释掉.bash_profile中的http_prosy, https_proxy, 然后在Terminal中执行  
```bash
	unset http_proxy  
	unset https_proxy  
	gem sources -a https://ruby.taobao.org  
```