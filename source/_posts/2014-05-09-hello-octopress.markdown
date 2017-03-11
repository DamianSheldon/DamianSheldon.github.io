---
layout: post
title: "Hello Octopress"
date: 2014-05-09 15:01:51 +0800
comments: true
categories: [Archives]
keywords: Octopress, blog, Problem, Solution
description: 用Octopress搭建自己博客过程中的常见问题及解决办法
---
> Octopress:A blogging framework for hackers.

看到介绍就被吸引了，正好最近想写些东西，于是决定用Octopress来搭个自己的博客。Octopress既然是为黑客准备的，有点难度那也是很正常的。网上很多都是介绍搭建的博客的步骤，但对出现各种问题以及解决办法的总结不是很多。而自己和ruby不是好朋友，这过程中出现很多的问题，这里把遇到的问题贴上来，一来做个总结；二来也可以给遇到同样的问题的朋友一些帮助。

#问题1(Update 2015.12.11)  
```bash
	An error occurred while installing RedCloth (4.2.9), and Bundler cannot continue.  
	Make sure that gem install RedCloth -v '4.2.9' succeeds before bundling.  
```

Solution:  

```
$ sudo gem install RedCloth -v '4.2.9' --verbose
```

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
<!-- more -->

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

##问题5(Update 2015.12.11)
怎么把文章放到navigation对应的分类组织中呢？

Solution:我在navigation中加了一个iOS Devlopment的分类是这么做的：
```bash
$ rake new_page[categories/iOS_Development]  
// Edit source/ios-development/index.html, file's eventually content is:
---
layout: default
title: "iOS Development"
---
// Note: % has been escaped
{\% for post in site.posts \%}
    {\% if post.categories contains 'ios development' \%}
        {\% include archive_post.html \%}
    {\% endif \%}
{\% endfor \%}

$ vim source/_includes/custom/navigation.html
// Add follow content
<li><a href="{{ root_url }}/blog/categories/ios-development">iOS Development</a></li>

// When you write a post add follow contents to YAML head:
categories: [iOS Development]
```

##问题6(Update 2014-11-22)
怎么把侧边栏放到左边去呢？

Solution:要想把侧边栏放到左边来，就得知道Octopress是如何布局的。Octopress是基于jkeyll。  
> Jekyll 的核心其实是一个文本转换引擎。它的概念其实就是：你用你最喜欢的标记语言来写文章，可以是 Markdown, 也可以是 Textile, 或者就是简单的 HTML, 然后 Jekyll 就会帮你套入一个或一系列的布局中。在整个过程中你可以设置 URL 路径，你的文本在布局中的显示样式等等。这些都可以通过纯文本编辑来实现，最终生成的静态页面就是你的成品了。

我们写的文章在source的_post目录下，每篇文章头部yaml信息指定转换的参数，其中layout就是布局的模板，sass目录下screen.scss是css信息的总入口，我们可以调整这些值得到我们想要的布局。


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

##问题 8
How clone your octopress to blog from two places?
Solution:

* Recreating a local Octopress repository

```
$ git clone -b source git@github.com:username/username.github.com.git octopress

$ cd octopress
$ git clone git@github.com:username/username.github.com.git _deploy 

```

* Pushing changes from two different machines

```
// From the first machine do the following whenever you’ve made changes:
$ rake generate
$ git add .
$ git commit -am "Some comment here." 
$ git push origin source  # update the remote source branch 
$ rake deploy             # update the remote master branch

// Then on the other machine, you need to pull those changes.
$ cd octopress
$ git pull origin source  # update the local source branch
$ cd ./_deploy
$ git pull origin master  # update the local master branch
```

Reference:[Clone Your Octopress to Blog From Two Places](http://blog.zerosharp.com/clone-your-octopress-to-blog-from-two-places/)  
