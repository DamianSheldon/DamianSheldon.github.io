---
layout: post
title: "Linux Useful Command Line Part 2"
date: 2015-09-11 09:30:16 +0800
comments: true
categories: [Archives]
keywords:
discription:
---
###1.Rails Error: ImageMagick/GraphicsMagick is not installed

```
brew install graphicsmagick
brew install ImageMagick
```
Reference:http://stackoverflow.com/questions/29377651/rails-error-imagemagick-graphicsmagick-is-not-installed

###2.Transfer file via scp (or sftp) between two Mac

Via scp:  
1.Enable Enable SSH
	 System Preferences > Sharing > enable the “Remote Login” feature

2.type scp command

```
scp VirtualBox-5.0.4-102546-OSX.dmg dongmeiliang@192.168.1.255:/Users/dongmeiliang/Documents/
```

Via sftp:  

1.sftp dongmeiliang@192.168.1.255  
2.cd path  
  Change remote directory to path.  

3.lcd path  
  Change local directory to path.  

4.get [-Ppr] remote-path [local-path]  

5.lpwd    Print local working directory.

6.pwd     Display remote working directory.

7.put [-Ppr] local-path [remote-path]

8.lls [ls-options [path]]
             Display local directory listing of either path or current directory if path is not specified.  ls-options may con-
             tain any flags supported by the local system's ls(1) command.  path may contain glob(3) characters and may match
             multiple files.

9.ls [-1afhlnrSt] [path]
             Display a remote directory listing of either path or the current directory if path is not specified.  path may con-
             tain glob(3) characters and may match multiple files.

10.Quit sftp

bye  
exit  
quit  

###3.Installing Apps with Homebrew Cask
[Homebrew Cask](http://caskroom.io/) is an extension for Homebrew that allows you to automate the installation of Mac Apps and Fonts.

After you have homebrew installed, you'll want to install Homebrew Cask:

```
$ brew install caskroom/cask/brew-cask
```

###4.复制目录及子目录下的某种类型文件

```
$ cp  ~/Documents/HJKApp/srsApp/srsApp/Sections/Home/Main.xcassets/**/*.png  ~/Pictures/HJKApp/OnlyHaveScale2/
```

###5.Recursive copy of specific files in Unix/Linux?

```
$ rsync -avm --include='*.jar' -f 'hide,! */' . /destination_dir
```

Reference:http://stackoverflow.com/questions/9622883/recursive-copy-of-specific-files-in-unix-linux

###6.Uploading directories with sftp?

I don't know why sftp does this but you can only recursive copy if the destination directory already exists. So do this...

```
sftp> mkdir bin  
sftp> put -r bin
```

Reference:http://unix.stackexchange.com/questions/7004/uploading-directories-with-sftp

###7.Scanning machine's specified ports

```
$ nmap -p 37099 192.168.3.214
// -Pn: Treat all hosts as online -- skip host discovery
$ nmap -Pn 192.168.3.214
```

###8.Reverse search history command

```
// Execute last command
$ !!

Ctrl + r
(reverse-i-search)`': pull
(reverse-i-search)`': git
```

Reference:http://unix.stackexchange.com/questions/3747/understanding-the-exclamation-mark-in-bash

###9.How to use rsync upload directory to remote server?
A:

```
$ rsync --delete --rsh=ssh -av /Users/dongmeiliang/Sites/upload/ meiliang@remote_ip:/home/meiliang/public_html/upload
```

