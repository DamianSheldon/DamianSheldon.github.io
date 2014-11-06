---
layout: post
title: "Git 小结"
date: 2014-11-05 10:13:13 +0800
comments: true
categories: [Archives]
keywords: Git, Summary
discription: This is a summary of git
---

###简介
git - the stupid content tracker  

这是man git中对它的介绍，我们看到它的核心是content tracker。Git是一个分布式的版本控制系统，项目是出于维护Linux内核源码的需求, 由Linus Torvalds启动的，现在已经成为最流行的版本管理系统，学会Git几乎成了开发者的必备技能。


###安装Git
安装Git主要有两种方法：一种是通过编译源代码来安装;另一种是使用为特定平台预编译好的安装包。  

###在服务器上布署Git
尽管技术上可以从个人的仓库里推送和拉取改变,但是我们不鼓励这样做,因为一不留心就很 容易弄混其他人的进度。另外,你也一定希望合作者们即使在自己不开机的时候也能从仓库获取数据——拥有 一个更稳定的公共仓库十分有用。因此,更好的合作方式是建立一个大家都可以访问的共享仓库,从那里推送和拉取数据。我们将把这个仓库称为 “Git 服务器”;代理一个 Git 仓库只需要花费很少的资源,几乎从不 需要整个服务器来支持它的运行。

架设一个 Git 服务器有很多种选择，这里不打算展开，不是一下子能讲清楚的，让我们看个实例。架设一个使用SSH传输数据和使用 authorized_keys 方法来给用户授权的Git服务器：
```bash
// Step 1:创建一个 ‘git’ 用户并为其创建一个 .ssh 目录
$ sudo adduser git 
$ su -l git
$ cd ~
$ mkdir .ssh

// Step 2:把开发者的 SSH 公钥添加到这个用户的 authorized_keys 文件中。
// 假设你通过 e-mail 收到了几个 公钥并存到了临时文件里
$ cat /tmp/id_rsa.john.pub >> ~/.ssh/authorized_keys
$ cat /tmp/id_rsa.josie.pub >> ~/.ssh/authorized_keys
$ cat /tmp/id_rsa.jessica.pub >> ~/.ssh/authorized_keys

// Step 3:使用 --bare 选项运行 git init 来设定一个空仓库,这会初始化一个不包含工作目录的仓库
$sudo mkdir /opt/git
$sudo chown -R git /opt/git
$ cd /opt/git
$ mkdir project.git 
$ cd project.git
$ git --bare init

// Mac 下需要开启ssh并允许remote login
$launchctl start sshd
// System Preferences -> Sharing -> Remote Login -> All Users

// Step 4:Join,Josie 或者 Jessica 就可以把它加为远程仓库,
推送一个分支,从而把第一个版本的工程上 传到仓库里了
# 在 John 的电脑上
$ cd myproject
$ git init
$ git add .
$ git commit -m 'initial commit'
$ git remote add origin git@gitserver:/opt/git/project.git 
$ git push origin master

// Step 5:其他人的克隆和推送也一样变得很简单
$ git clone git@gitserver:/opt/git/project.git $ vim README
$ git commit -am 'fix for the README file'
$ git push origin master

// Step 6:用这个方法可以很快捷的为少数几个开发者架设一个可读写的 Git 服务


// 如何生成 SSH 公钥？
// Step 1:首先,确定一下是否已经有一个公钥了。SSH 公钥默认储存 在账户的 ~/.ssh 目录。
// 进入那里并查看其内容,有没有公钥一目了然:
$ cd ~/.ssh
$ ls
authorized_keys2 id_dsa known_hosts config id_dsa.pub

// Step 2:关键是看有没有用 文件名 和 文件名.pub 来命名的一对文件,
// 这个 文件名 通常是 id_dsa 或者 id_rsa。 .pub 文件是公钥,另一个文件是密钥。
// 假如没有这些文件(或者干脆连 .ssh 目录都没有),你可以用 ssh- keygen 的程序来建立它们,
// 该程序在 Linux/Mac 系统由 SSH 包提供, 在 Windows 上则包含在 MSysGit 包 里:
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/schacon/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /Users/schacon/.ssh/id_rsa.
Your public key has been saved in /Users/schacon/.ssh/id_rsa.pub.
The key fingerprint is:
43:c5:5b:5f:b1:f1:50:43:ad:20:a6:92:6a:1f:9a:3a schacon@agadorlaptop.local

// Step 3:它先要求你确认保存公钥的位置(.ssh/id_rsa),然后它会让你重复一个密码两次,
// 如果不想在使用公钥的 时候输入密码,可以留空。
// 现在,所有做过这一步的用户都得把它们的公钥给你
// 或者 Git 服务器的管理者(假设 SSH 服务被设定为使 用公钥机制)。
// 他们只需要复制 .put 文件的内容然后 e-email 之。公钥的样子大致如下:
$ cat ~/.ssh/id_rsa.pub
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAklOUpkDHrfHY17SbrmTIpNLTGK9Tjom/BWDSU 
GPl+nafzlHDTYW7hdI4yZ5ew18JH4JW9jbhUFrviQzM7xlELEVf4h9lFX5QVkbPppSwg0cda3 
Pbv7kOdJ/MTyBlWXFCR+HAo3FXRitBqxiX1nKhXpHAZsMciLq8V6RjsNAQwdsdMFvSlVK/7XA 
t3FaoJoAsncM1Q9x5+3V0Ww68/eIFmb1zuUFljQJKprrX88XypNDvjYNby6vw/Pb0rwert/En 
mZ+AW4OZPnTPI89ZPmVMLuayrD2cE86Z/il8b+gw3r3+1nKatmIkjn2so1d01QraTlMqVSsbx 
NrRFi9wrf+M7Q== schacon@agadorlaptop.local
```

作为一个额外的防范措施,你可以用 Git 自带的 git-shell 简单工具来把 git 用户的活动限制在仅与 Git 相关。把它设为 git 用户登入的 shell,那么该用户就不能拥有主机正常的 shell 访问权。为了实现这一 点,需要指明用户的登入shell 是 git-shell ,而不是 bash 或者 csh。你可能得编辑 /etc/passwd 文件:
```bash
// Step 1:
$ sudo vim /etc/passwd

// Step 2:在文件末尾,你应该能找到类似这样的行
git:x:1000:1000::/home/git:/bin/sh

// Step 3:把 bin/sh 改为 /usr/bin/git-shell (或者用 which git-shell 查看它的位置)。该行修改后的样子如下
git:x:1000:1000::/home/git:/usr/bin/git-shell

// Step 4:现在 git 用户只能用 SSH 连接来推送和获取 Git 仓库,而不能直接使用主机 shell。
// 尝试登录的话,你会 看到下面这样的拒绝信息
$ ssh git@gitserver
fatal: What do you think I am? A shell? (你以为我是个啥?shell吗?) 
Connection to gitserver closed. (gitserver 连接已断开。)
```
###Git基础
为了和其他人愉快地的合作开发，我们要掌握Git的基本命令。你不会想看到人民群众仇恨你。    
1)Git基础要点;   
2)配置Git;   
3)Git分支;
4)记录每次更新到仓库;    
5)撤消操作;   
6)远程仓库的使用;    
7)打标签;    
8)查看提交历史。    

####Git基础要点
对于任何一个文件,在 Git 内都只有三种状态:已提交 (committed),已修改(modified)和已暂存(staged)。已提交表示该文件已经被安全地保存在本地数据 库中了;已修改表示修改了某个文件,但还没有提交保存;已暂存表示把已修改的文件放在下次提交时要保存 的清单中。
由此我们看到 Git 管理项目时,文件流转的三个工作区域:Git 的本地数据目录,工作目录以及暂存区域。

每个项目都有一个 git 目录,它是 Git 用来保存元数据和对象数据库的地方。该目录非常重要,每次克隆 镜像仓库的时候,实际拷贝的就是这个目录里面的数据。
从项目中取出某个版本的所有文件和目录,用以开始后续工作的叫做工作目录。这些文件实际上都是从 git 目录中的压缩对象数据库中提取出来的,接下来就可以在工作目录中对这些文件进行编辑。
所谓的暂存区域只不过是个简单的文件,一般都放在 git 目录中。有时候人们会把这个文件叫做索引文件,不过标准说法还是叫暂存区域。

基本的 Git 工作流程如下:    
a)在工作目录中修改某些文件;    
b)对这些修改了的文件作快照,并保存到暂存区域;    
c)提交更新,将保存在暂存区域的文件快照转储到 git 目录中.   

#####配置Git

一般在新的系统上,我们都需要先配置下自己的 Git 工作环境。配置工作只需一次,以后升级时还会沿用 现在的配置。当然,如果需要,你随时可以用相同的命令修改已有的配置。

Git 提供了一个叫做 git config 的工具(译注:实际是 git-config 命令,只不过可以通过 git 加一个 名字来呼叫此命令。),专门用来配置或读取相应的工作环境变量。而正是由这些环境变量,决定了 Git 在 各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方:    
1)/etc/gitconfig文件:系统中对所有用户都普遍适用的配置。若使用 git config 时用 --system 选项,读写 的就是这个文件。    
2)~/.gitconfig文件:用户目录下的配置文件只适用于该用户。若使用 git config 时用 --global 选项,读写 的就是这个文件。   
3)当前项目的 git 目录中的配置文件(也就是工作目录中的 .git/config 文件):这里的配置仅仅针对当前 项目有效。每一个级别的配置都会覆盖上层的相同配置,所以 .git/config 里的配置会覆盖 /etc/gitconfig 中的同名变量。   

```bash
// Quilk config
// 用户信息
$ git config --global user.name "John Doe"
$ git config --global user.email johndoe@example.com

// 文本编辑器
$ git config --global core.editor emacs

// 差异分析工具
$ git config --global merge.tool vimdiff

// 查看配置信息
$ git config --list

```

#####Git分支
几乎每一种版本控制系统都以某种形式支持分支。使用分支意味着你可以从开发主线上分离开来,然后在不影响主线的同时继续工作。
1)新建分支;
```bash
// 新建本地分支
git branch [--set-upstream | --track | --no-track] [-l] [-f] <branchname> [<start-point>]

--set-upstream
如果指定的分支不存在，或者指定了--force参数，作用和--track一样。
否则当创建分支时建立像--track一样的配置，除了分支的指向没有改变。

--track

当创建新的分支，从新分支建立branch.<name>.remote和branch.<name>.merge的配置入口
去标记start-point 分支作为“upstream”。
这个配置会告诉git在git status和git branch -v中显示两个分支的关系。
而且，当新的分支被检出时，它会引导git pull在没带参数时去从upstream拉代码。

当我们的start point是remote-tracking分支时，这一行为是默认的。
如果你想让git checkout和git branch总是像给定--no-track一样执行，设置branch.autosetupmerge 配置变量为false。
当start-point是local或remote-tracking分支时，上述行为是你想要的，那么设置它为always。

--no-track
不建立"upstream"配置，即使branch.autosetupmerge配置变量的值是true。

-l
创建分支的引用日志。它会激活对分支引用所有改变模式的记录，开启使用基于sha1表达式的日期，
如"<branchname>@{yesterday}"。
注意在non-bare仓库中，引用日志由于core.logallrefupdates配置选项默认都开启的。


-f
如果<branchname>已经存在，重置<branchname>到<startpoint>。没有-f git branch将会拒绝改变。

<branchname>
要创建或删除分支的名称。新分支的名称必须通过git-check-ref-format(1)定义的所有检查。
有些检查会限制分支名称中能使用的字符。

<start-point>
已经存在的分支名，对它应用和<branchname>相同的限制。

如果我们没有指定<start-point>,它默认是HEAD。
$git branch <branchname> <start-point>

// 新建远程仓库分支
git push [远程名] [本地分支]:[远程分支]

```

2)删除分支;
```bash
// 删除本地分支
$ git branch -d <branchname>

// 删除远程分支
// 如果想在服务器上删 除 serverfix 分支,运行下面的命令:
$ git push origin :serverfix
To git@github.com:schacon/simplegit.git
- [deleted] serverfix
```

3)切换分支;
```bash
git checkout <branchname>
```

4)合并分支;
```bash
// 将sourceBranchname合并到destinationBranchname
$git checkout destinationBranchname
$git merge sourceBranchname
```

5)衍合分支。
```bash
// 把一个分支整合到另一个分支的办法有两种:merge(合并) 和 rebase(衍合)。
// 把在 branchA 里产生的变化补丁重新在 branchB 的基础上打一遍。在 Git 里,这种 操作叫做衍合(rebase)。
// 有了 rebase 命令,就可以把在一个分支里提交的改变在另一个分支里重放一遍。
// 例如将experiment衍合到master分支:
$ git checkout experiment
$ git rebase master
First, rewinding head to replay your work on top of it... Applying: added staged command

```
关于如何进行分支管理，可以看看阮一峰老师的[这篇博文](http://www.ruanyifeng.com/blog/2012/07/git.html)。

#####记录每次更新到仓库
版本控制的主要作用就是记录我们的更新，如果我们不将更新记录到远程仓库就失去意义了。
```bash
// 检查当前文件状态
$ git status
# On branch master
nothing to commit (working directory clean)

// 跟踪新文件
$ git add newFilename

// 暂存已修改文件
$ git add trackedFilename

// 忽略某些文件
//一般我们总会有些文件无需纳入 Git 的管理,也不希望它们总出现在未跟踪文件列表。
通常都是些自动生成的文件,像是日志或者编译过程中创建的等等。
我们可以创建一个名为 .gitignore 的文件,列出要忽略的 文件模式。

// 文件 .gitignore 的格式规范如下:
// 1)所有空行或者以注释符号 # 开头的行都会被 Git 忽略。
// 2)可以使用标准的 glob 模式匹配, 所谓的 glob 模式是指 shell 所使用的简化了的正则表达式。
// 3)匹配模式最后跟反斜杠(/)说明要忽略的是目录。
/*
# 此为注释 – 将被 Git 忽略
*.a # 忽略所有 .a 结尾的文件
!lib.a # 但 lib.a 除外
/TODO # 仅仅忽略项目根目录下的 TODO 文件,不包括 subdir/TODO build/ # 忽略 build/ 目录下的所有文件
doc/*.txt # 会忽略 doc/notes.txt 但不包括 doc/server/arch.txt
*/

// 查看已暂存和未暂存的更新
// 实际上 git status 的显示比较简单,仅仅是列出了修改过的文件,如果要查看具体修改了什么地方,可以用 git diff 命令。
$ git diff
diff --git a/benchmarks.rb b/benchmarks.rb index 3cb747f..da65585 100644
--- a/benchmarks.rb
+++ b/benchmarks.rb
@@ -36,6
+ + + +
+36,10 @@ def main @commit.parents[0].parents[0].parents[0]
end
run_code(x, 'commits 1') do git.commits.size
end
run_code(x, 'commits 2') do
log = git.commits('master', 15) log.size

// 若要看已经暂存起来的文件和上次提交时的快照之间的差异,可以用 git diff --cached 命令。

// 提交更新
$ git commit

// 跳过使用暂存区域
// 尽管使用暂存区域的方式可以精心准备要提交的细节,但有时候这么做略显繁琐。
Git 提供了一个跳过使用 暂存区域的方式,只要在提交的时候,给 git commit 加上 -a 选项,
Git 就会自动把所有已经跟踪过的文件暂存起来一并提交,从而跳过 git add 步骤:
$ git commit -a -m 'added new benchmarks'
[master 83e38c7] added new benchmarks
1 files changed, 5 insertions(+), 0 deletions(-)

// 移除文件
// 要从 Git 中移除某个文件,就必须要从已跟踪文件清单中移除(确切地说,是从暂存区域移除),然后提交。
可以用 git rm 命令完成此项工作,并连带从工作目录中删除指定的文件,这样以后就不会出现在未跟踪 文件清单中了。
// 如果只是简单地从工作目录中手工删除文件,运行 git status 时就会在 “Changed but not updated” 部分(也就是_未暂存_清单)看到。

// 移动文件
$ git mv file_from file_to

```

#####撤消操作
任何时候,你都有可能需要撤消刚才所做的某些操作。
```bash
// 修改最后一次提交
// 有时候我们提交完了才发现漏掉了几个文件没有加,或者提交信息写错了。想要撤消刚才的提交操作,可以 使用 --amend 选项重新提交:
$ git commit --amend

// 取消已经暂存的文件
git reset HEAD <file>...

// 取消对文件的修改
use "git checkout -- <file>..." to discard changes in working directory


```

#####远程仓库的使用
```bash
// 查看当前的远程库
git remote [-v]

// 添加远程仓库
git remote add [shortname] [url]

// 从远程仓库抓取数据
$ git fetch [remote-name]

// 推送数据到远程仓库
git push [remote-name] [branch-name]

// 查看远程仓库信息
git remote show [remote-name]

// 远程仓库的删除和重命名
// 可以用 git remote rename 命令修改某个远程仓库的简短名称,比如想把 pb 改成 paul,可以这么运行:
$ git remote rename pb paul $ git remote
origin
paul

// 移除 对应的远端仓库,可以运行 git remote rm 命令:
$ git remote rm paul $ git remote
origin
```

#####打标签
人们在发布某个软件版本(比如 v1.0 等等)的时候,经常会打上一标签。
```bash
// 列显已有的标签
$ git tag v0.1
v1.3

// 新建标签
// Git 使用的标签有两种类型:轻量级的(lightweight)和含附注的(annotated)。
轻量级标签就像是个不 会变化的分支,实际上它就是个指向特定提交对象的引用。
而含附注标签,实际上是存储在仓库中的一个独立 对象,它有自身的校验和信息,
包含着标签的名字,电子邮件地址和日期,以及标签说明,标签本身也允许使 用 GNU Privacy Guard (GPG) 来签署或验证。
一般我们都建议使用含附注型的标签,以便保留相关信息;
当然,如果只是临时性加注标签,或者不需要旁注额外信息,用轻量级标签也没问题。

// 创建一个含附注类型的标签非常简单,用 -a (译注:取 annotated 的首字母)指定标签名字即可:
$ git tag -a v1.4 -m 'my version 1.4'

// 可以使用 git show 命令查看相应标签的版本信息,并连同显示打标签时的提交对象。
$ git show v1.4
tag v1.4
Tagger: Scott Chacon <schacon@gee-mail.com> Date: Mon Feb 9 14:45:11 2009 -0800
my version 1.4
commit 15027957951b64cf874c3557a0f3547bd83b3ff6 Merge: 4a447f7... a6b4c97...
Author: Scott Chacon <schacon@gee-mail.com> Date: Sun Feb 8 19:02:46 2009 -0800
Merge branch 'experiment'

// 签署标签
//如果你有自己的私钥,还可以用 GPG 来签署标签,只需要把之前的 -a 改为 -s (译注: 取 Signed 的首
字母)即可:
$ git tag -s v1.5 -m 'my signed 1.5 tag'
You need a passphrase to unlock the secret key for user: "Scott Chacon <schacon@gee-mail.com>" 
1024-bit DSA key, ID F721C45A, created 2009-02-09

// 轻量级标签
// 轻量级标签实际上就是一个保存着对应提交对象的校验和信息的文件。
要创建这样的标签,一个 -a,-s 或 -m 选项都不用,直接给出标签名字即可:
$ git tag v1.4-lw

// 验证标签
// 可以使用 git tag -v [tag-name] (译注:取 verify 的首字母)的方式验证已经签署的标签。
此命令会调用 GPG 来验证签名,所以你需要有签署者的公钥,存放在 keyring 中,才能验证:
$ git tag -v v1.4.2.1
object 883653babd8ee7ea23e6a5c392bb739348b1eb61
type commit
tag v1.4.2.1
tagger Junio C Hamano <junkio@cox.net> 1158138501 -0700
GIT 1.4.2.1
Minor fixes since 1.4.2, including git-mv and git-http with alternates.
gpg: Signature made Wed Sep 13 02:08:25 2006 PDT using DSA key ID F3119B9A 
gpg: Good signature from "Junio C Hamano <junkio@cox.net>"
gpg: aka "[jpeg image of size 1513]"
Primary key fingerprint: 3565 2A26 2040 E066 C9A7 4A7D C0C6 D9A4 F311 9B9A

// 后期加注标签
// 比如在下面展示的提交历史中:
$ git log --pretty=oneline 
15027957951b64cf874c3557a0f3547bd83b3ff6 Merge branch 'experiment' 
a6b4c97498bd301d84096da251c98a07c7723e65 beginning write support 
0d52aaab4479697da7686c15f77a3d64d9165190 one more thing 
6d52a271eda8725415634dd79daabbc4d9b6008e Merge branch 'experiment' 
0b7434d86859cc7b8c3d5e1dddfed66ff742fcbc added a commit function 
4682c3261057305bdd616e23b64b0857d832627b added a todo file 
166ae0c4d3f420721acbb115cc33848dfcc2121a started write support 
9fceb02d0ae598e95dc970b74767f19372d61af8 updated rakefile 
964f16d36dfccde844893cac5b347e7b3d44abbc commit the todo 
8a5cbc430f1a9c3d00faaeffd07798508422908a updated readme

// 我们忘了在提交 “updated rakefile” 后为此项目打上版本号 v1.2,没关系,现在也能做。
只要在打标 签的时候跟上对应提交对象的校验和(或前几位字符)即可:
$ git tag -a v1.2 9fceb02

// 分享标签
// 默认情况下,git push 并不会把标签传送到远端服务器上,只有通过显式命令才能分享标签到远端仓库。
其命令格式如同推送分支,运行 git push origin [tagname] 即可:
$ git push origin v1.5
Counting objects: 50, done.
Compressing objects: 100% (38/38), done. Writing objects: 100% (44/44), 4.56 KiB, done. 
Total 44 (delta 18), reused 8 (delta 1)
To git@github.com:schacon/simplegit.git
* [new tag] v1.5 -> v1.5

// 如果要一次推送所有(本地新增的)标签上去,可以使用 --tags 选项:
$ git push origin --tags
```

#####查看提交历史
```bash
// 在提交了若干更新之后,又或者克隆了某个项目,想回顾下提交历史,可以使用 git log 命令。

// 我们常用 -p 选项展开显示每次提交的内容差异,用 -2 则仅显示最近的两次更新:
$ git log –p -2

// 还有 许多摘要选项可以用,比如 --stat,仅显示简要的增改行数统计:
$git log --stat

// 限制输出长度
// 列出所有最近两周内的提交
$ git log --since=2.weeks

```


###分布式工作流程
在服务器上布暑好了Git，并建好代码仓库以后，团队成员就可以愉快地合作开发了。由于团队的规模不一样，工作流程也会略有区别，我们先通过最简单的私有的小型团队来掌握基本的流程，其他的情况也就容易理解了。
```bash
//一个私有项目,与你一起协作的还有另外一到两位开发者。这里说私有,是指源代码不公开,其他人无法访问项目仓库。

# John's Machine
// Step 1: 克隆一份项目代码到本地
$ git clone john@githost:simplegit.git
Initialized empty Git repository in /home/john/simplegit/.git/ ...

// Step 2: 打开项目，编辑，完成属于自己的任务
$ cd simplegit/
$ vim lib/simplegit.rb

// Step 3:更新项目，因为在你完成任务的时间窗口中团队其他成员可能提交过代码
$ git fetch origin
...
From john@githost:simplegit
+ 049d078...fbff5bc master -> origin/master

// Step 4:合并分支，将团队成员的代码和自己的代码合并到一起
$ git merge origin/master Merge made by recursive.
TODO | 1 +
1 files changed, 1 insertions(+), 0 deletions(-)

// Step 5:合并分支冲突时
$git status

index.html: needs merge
# # # # # # #
On branch master
Changed but not updated:
(use "git add <file>..." to update what will be committed)
(use "git checkout -- <file>..." to discard changes in working directory)
unmerged: index.html

// 任何包含未解决冲突的文件都会以未合并(unmerged)状态列出。
Git 会在有冲突的文件里加入标准的冲突 解决标记,可以通过它们来手工定位并解决这些冲突。
可以看到此文件包含类似下面这样的部分:
<<<<<<< HEAD:index.html
<div id="footer">contact : email.support@github.com</div> 
=======
<div id="footer">
please contact us at support@github.com </div>
>>>>>>> iss53:index.html

// 可以看到 ======= 隔开的上半部分,是 HEAD(即 master 分支,在运行 merge 命令时检出的分支)中的内 容,
下半部分是在 iss53 分支中的内容。解决冲突的办法无非是二者选其一或者由你亲自整合到一起。
比如你可以通过把这段内容替换为下面这样来解决:
<div id="footer">
please contact us at email.support@github.com </div>

// Step 6:手动解决冲突，然后运行 git add 将把它们标记为已解决(resolved),
如果觉得满意了,并且确认所有冲突都已解决,也就是进入了缓存区,就可以用 git commit 来完成这次合并提交。


$ git commit -am 'removed invalid default value'
[master 738ee87] removed invalid default value
1 files changed, 1 insertions(+), 1 deletions(-)

// Step 7:将完成的代码推送到服务器的代码仓库中
$ git push origin master
...
To jessica@githost:simplegit.git
1edee6b..fbff5bc master -> master

```

####Reference
[Pro Git](http://git-scm.com/book/zh/v1)