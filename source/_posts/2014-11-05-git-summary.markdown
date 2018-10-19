---
layout: post
title: "Git 小结"
date: 2014-11-05 10:13:13 +0800
comments: true
categories: [Archives]
keywords: Git, Summary
description: A summary of basic usage of git
---
本文是从日常开发的角度对 Git 的总结，主要内容如下：

1. 搭建 Git 服务器
2. 初始化远程仓库
3. 多人协作开发
4. 提交测试
5. 修复 BUG

## 搭建 Git 服务器

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

// Preparation & Accounts Setup
$ sudo useradd -G git -d /home/john -m -s /bin/bash john
$ sudo useradd -G git -d /home/andrew -m -s /bin/bash andrew
$ sudo useradd -G git -d /home/robert -m -s /bin/bash robert

// Exist user change his primary group by newgrp command
$ sudo newgrp -g git john

// Step 3:使用 --bare 选项运行 git init 来设定一个空仓库,这会初始化一个不包含工作目录的仓库
$sudo mkdir /git
$sudo chown -R git /git
$ cd /git
$ mkdir project.git 
$ cd project.git
$ git --bare --shared=group init

// if forget init with --shared=group, we can config with command:
$ git config -- core.sharedRepository ture

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
$ git remote add origin git@gitserver:/git/project.git 
$ git push origin master

// Step 5:其他人的克隆和推送也一样变得很简单
$ git clone git@gitserver:/git/project.git 
$ vim README
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
<!-- more -->

## 初始化远程仓库

```
$ su - git
$ cd /git
$ git init --bare sample.git
```

## 多人协作开发
1. 场景一：新建工程

```
$ git clone git@server:/git/sample.git
$ git checkout -b develop
$ add .gitignore （1）
$ new project
$ git add .
$ git commit -m "Description what you have done."
$ git push origin develop:develop --verbose
```

2. 场景二：正常日常开发

```
$ develop ...
$ git add .
$ git commit -m "Description what you have done." 
$ git pull origin develop
$ fix conflict if need
$ git push origin develop:develop --verbose 
```

3. 场景三：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改

```
$ git checkout -- file
```

4. 场景四：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改

```
$ git reset HEAD file
$ git checkout -- file
```

5. 场景五：已经提交了不合适的修改到版本库时，想要撤销本次提交，没有推送到远程库

```bash
# 回到过去某个版本
$ git log
$ git reset --hard commit_id
# --hard 会让工作区的内容也撤销，如果我们想保留工作区的内容则不需要加--hard
# 例如新建工程时，忘了先加gitignore 文件，这时我们便可以使用它回滚索引记录，
# 添加 gitignore 文件后重新添加后再提交。
$ git reset commit_id 

# 回到过去某个版本之后又想返回未来的版本
$ git reflog
$ git reset --hard commit_id
```

6. 场景六：已经提交了不合适的修改到远程库，想要撤销本次提交

```
$ git rm --cached <file>
```

7. 场景七：修改提交信息

```
// Change the last commit
$ git commit --amend

// Change published commit message
$ git rebase -i HEAD~3

```

8. 场景八：查看修改

```
// 查看文件做了哪些修改
$ git diff <commit> <commit> file

// 先查看同事做了哪些修改，然后再合并
$ git fetch origin develop --verbose
$ git diff HEAD FETCH_HEAD
$ git merge origin/develop

// Various ways to check your working tree
$ git diff            (1)
$ git diff --cached   (2)
$ git diff HEAD       (3)
```
1. Changes in the working tree not yet staged for the next commit.
2. Changes between the index and your last commit; what you would be committing if you run "git commit" without "-a" option.
3. Changes in the working tree since your last commit; what you would be committing if you run "git commit -a"

```
// Comparing with arbitrary commits
$ git diff test            (1)
$ git diff HEAD -- ./test  (2)
$ git diff HEAD^ HEAD      (3)
```
 1. Instead of using the tip of the current branch, compare with the tip of "test" branch.
2. Instead of comparing with the tip of "test" branch, compare with the tip of the current branch, but limit the comparison to the file "test".
3. Compare the version before the last commit and the last commit.

9. 场景九：打标签归档

```
// 创建标签
$ git tag <name>
$ git tag -a <tagname> -m "blablabla..."

// 查看标签
$ git tag
$ git show <tagname>

// 推送标签
$ git push origin <tagname>
$ git push origin develop:develop --tags --verbose

// 删除标签
$ git tag -d <tagname>

// 删除一个远程标签
$ git tag -d <tagname>
$ git push origin :refs/tags/<tagname>
```

10. 场景十：有的命令太长记不住，配置别名

```
// 配置别名
$ git config --global alias.last 'log -1'
$ git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"

// 使用别名
$ git last
$ git lg

// 查看配置了哪些别名
$ cat ~/.gitconfig

// 删除别名
vim ~/.gitconfig > 别名就在[alias]后面，要删除别名，直接把对应的行删掉即可
```

## 提交测试
1. 场景一：尚未创建 test 分支

```
$ git checkout -b test 
$ git push origin develop:test --verbose
```

2. 场景二：已创建 test 分支

```
$ git pull origin test --verbose
$ fix conflict if need
$ git push origin test:test --verbose
```

## 修复 BUG
1. 场景一：开发完成后测试反馈 BUG

```
$ git checkout develop
$ fix bug 
$ git pull origin develop --verbose
```

2. 场景二：开发新功能中反馈紧急 BUG, 需中断开发工作，但开发工作只进行到一半，还没法提交

```
$ git stash
$ git checkout -b hotfix tag (1)
$ git push origin hotfix:develop --verbose
$ git checkout develop
$ git stash list
$ git stash pop (or git stash apply stash_identifier)
$ git branch -d hotfix
```

##Git 基本操作

### 操作 repository

```
// 1. Create a new repository
$ git init --bare sample.git

// 2. Addding a existing project to git
1)Initialize the local directory as a Git repository.
$ git init

2)Add the files in your new local repository. This stages them for the first commit.
$ git add .

3)Commit the files that you've staged in your local repository.
$ git commit -m "First commit"

4)In Terminal, add the URL for the remote repository where your local repository will be pushed.
// Sets the new remote
$ git remote add origin remote repository URL

// Verifies the new remote URL
$ git remote -v

5)Push the changes in your local repository to GitHub.
// Pushes the changes in your local repository up to the remote repository you specified as the origin
$ git push -u origin master
```

### 操作 branch

```
// List all existing branches
$ git branch

// Switch HEAD branch
$ git checkout <branch>

// Create a new branch based on your current HEAD
$ git branch <new-branch>

// Create a new tracking branch based on a remote branch
$ git branch --track <new-branch> <remote-branch>

// Delete a local branch
$ git branch -d <branch>

// Delete a remote branch
$ git push origin --delete <branch>

// Rename a branch locally
$ git branch -m <old name> <new name>

// Rename a branch on remote
$ git push <remote> :<old name>
$ git push <remote> <new name>
```

### 操作 file

```
// Add all current changes in file to the next commit 
$ git add <file>

// Delete file and add its deletion to next commit
$ git rm <file>

// Rename file and add it to next commit
$ git mv <file> <new file name>

// Show changes over time for a specific file
$ git log -p <file>

// Who changed what and when in file
$ git blame <file>

// Remove file from all previous commits but keep it locally
// 如何不再控制已经提交到仓库的文件？
$ git rm --cached <file>

// Manually resolve conflicts using your editor and mark file as resolved
$ git add <resolved-file>
$ git rm <resolved-file>

// Discard local changes in a specific file
$ git checkout HEAD <file>

// Revert a commit by providing a new commit with contrary changes
$ git revert <commit>

// Restore a specific file from a previous commit
$ git checkout <commit> <file>

// 如何将另一分支上的某文件复制到当前分支上来？
$ git checkout currentBranch
$ git checkout anotherBranch -- filenameYourWantedCopy

// 不再跟踪已经删除的文件
$ git add --update

// Shows the commits that changed specific file
$ git log --follow builtin/rev-list.c
// Shows the commits that changed builtin/rev-list.c, including those commits that occurred before the file was given its present name.

// Viewing a Deleted File
$ git show HEAD^:path/to/file

// You can use an explicit commit identifier or HEAD~n to see older versions or if therehas been more than one commit since you deleted it.

// Finally, most commands that take filenames will optionally allow you to precede any
// filename by a commit, to specify a particular version of the file:
$ git diff v2.5:Makefile HEAD:Makefile.in

// You can also use git show to see any such file:
$ git show v2.5:Makefile

Reference:[Viewing a Deleted File in Git](http://stackoverflow.com/questions/1395445/viewing-a-deleted-file-in-git)

// How to take a file out of another commit
// Example 1
$ git checkout master~2 Makefile

// Example 2
$ git checkout 4280f4a14319752308007124cb2a15fffd696025 Networking.podspec

```

### 操作 commit

```
// Commit previously staged changes
$ git commit

// Show all commits
$ git log

// Show changes over time for a specific committer
$ git log --author=<committer name>

// Change the last commit
$ git commit -amend

// Change published commit message
$ git rebase -i HEAD~3
```

##问题汇总

###1.如何将目录覆盖下一级的同名目录？
```
// 背景
|-A
|-B
  |-A
  
// 目标
|-B
  |-A
  
// 解决办法
1. rsync -a A/ B/A
2. (cd A && tar c .) | (cd B/A && tar xf -)

```

###2.insufficient permission for adding an object to repository database ./objects

```
ssh me@myserver
cd repository.git

sudo chmod -R g+w *

git config core.sharedRepository true

```

Reference:http://stackoverflow.com/questions/1918524/error-pushing-to-github-insufficient-permission-for-adding-an-object-to-reposi

###3.warning: remote HEAD refers to nonexistent ref, unable to checkout

Solution:问题的原因是远程仓库的默认分支名不为master。

```
// Login to remote server
git branch -a 
git branch -m branchName master

// local
git clone .... // Now the warn will be disappear

git fetch 
git add .
git commit
git push ...

```
Reference:http://blog.csdn.net/gracioushe/article/details/6142793

###4.How to untrack all delete files?

```
$ git diff-files --diff-filter=D --name-only -z | xargs -0 git rm
```

###5.Resolving a 'both added' merge conflict in git?

A: If you use git rm git will remove all versions of that path from the index so your resolve action will leave you without either version.

You can use git checkout --ours file to chose the version from the branch onto which you are rebasing or git checkout --theirs file to chose the version from the branch which you are rebasing.

If you want a blend you need to use a merge tool or edit it manually.

Reference:http://stackoverflow.com/questions/9823692/resolving-a-both-added-merge-conflict-in-git

###6.Do a “git export” (like “svn export”)?
A:git archive master | tar -x -C /somewhere/else

Reference:http://stackoverflow.com/questions/160608/do-a-git-export-like-svn-export

###7.部分远程分支在本地未显示
A:

```
$ git remote update
$ git branch -r
```
Reference:[Remote branch not showing up in “git branch -r”](https://stackoverflow.com/questions/12319968/remote-branch-not-showing-up-in-git-branch-r)

###8.如何删除已经提交到仓库的大文件或敏感文件？
A:Pro Git 的 Git Internals > Maintenance and Data Recovery > Removing Objects 给出问题的解决办法。


####Reference
[Pro Git](http://git-scm.com/book/zh/v1)

