---
layout: post
title: "Linux 使用笔记(二)"
date: 2015-09-11 09:30:16 +0800
comments: true
categories: [Archives]
keywords: Linux, command line
description: Linux usage notes
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
<!-- more -->
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
[Homebrew Cask](https://brew.sh/) is an extension for Homebrew that allows you to automate the installation of Mac Apps and Fonts.

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

###10.Recursively Search All Files For A String

```
// grep command: Recursively Search All Files For A String

cd /path/to/dir
grep -r "word" .

grep -r "string" .

// Ignore case distinctions:
grep -ri "word" .

// To display print only the filenames with GNU grep, enter:
grep -r -l "foo" .

// You can also specify directory name:
grep -r -l "foo" /path/to/dir/*.c

// find command: Recursively Search All Files For A String

// find command is recommend because of speed and ability to deal with filenames that contain spaces.
cd /path/to/dir
find . -type f -exec grep -l "word" {} +
find . -type f -exec grep -l "seting" {} +
find . -type f -exec grep -l "foo" {} +
// Older UNIX version should use xargs to speed up things:
find /path/to/dir -type f | xargs grep -l "foo"

// It is good idea to pass -print0 option to find command that it can deal with filenames that contain spaces or other metacharacters:
find /path/to/dir -type f -print0 | xargs -0 grep -l "foo"

```

###11. Common terminal shutcuts

```
Ctrl + A    Go to the beginning of the line you are currently typing on
Ctrl + E    Go to the end of the line you are currently typing on
Ctrl + L    Clears the Screen, similar to the clear command
Ctrl + U    Clears the line before the cursor position. If you are at the end of the line, clears the entire line.
Ctrl + H    Same as backspace
Ctrl + R    Let’s you search through previously used commands
Ctrl + C    Kill whatever you are running
Ctrl + D    Exit the current shell
Ctrl + Z    Puts whatever you are running into a suspended background process. fg restores it.
Ctrl + W    Delete the word before the cursor
Ctrl + K    Clear the line after the cursor
Ctrl + T    Swap the last two characters before the cursor
Esc + T Swap the last two words before the cursor
```

###12. 比较目录下同名文件

```
$ diff -Naur old-dir new-dir
```

###13. curl Command Download File Example

A:

```
$ curl  -o LongM4A.m4a http://download.wavetlan.com/SVV/Media/HTTP/LongM4A.m4a
```
Reference:http://www.cyberciti.biz/faq/curl-download-file-example-under-linux-unix/

###13. How to Convert mp3 to aac?
A: We can use ffmpeg archive this.

```
$ ffmpeg -i input.mp3 -c:a aac output.m4a
```

Reference:[FFmpeg command to convert MP3 to AAC](http://superuser.com/questions/370625/ffmpeg-command-to-convert-mp3-to-aac)

###14. How to enable tab to complete for new users
A:Sounds like you created this user with the `useradd` command. This is a low level command, and not the prefered way to create a user. By default it will point the user's shell to /bin/sh, not /bin/bash. So you will not get the nice autocompletion. You can check user's shell with command ` echo $SHELL`, list all available shells `cat /etc/shells`, change the user's shell:

```
$ sudo usermod --shell /bin/bash [username]
```

This should give you bash autocompletion. You probably also want to copy /etc/skel/.bashrc to the user's home directory.

In the future, prefer the `adduser` command. Despite the similar name, it has different results. It will default to using /bin/bash as the shell and seed the new user's home directory with the files from /etc/skel/

Reference:[How to enable tab to complete for new users in ubuntu](https://www.digitalocean.com/community/questions/how-to-enable-tab-to-complete-for-new-users-in-ubuntu)  
[3 Ways to Change a Users Default Shell in Linux](https://www.tecmint.com/change-a-users-default-shell-in-linux/)  


