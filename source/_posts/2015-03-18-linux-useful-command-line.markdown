---
layout: post
title: "Linux 命令行(一)"
date: 2015-03-18 21:12:56 +0800
comments: true
categories: [Archives]
keywords: Linux Useful Command Line
discription: Linux Useful Command Line
---

###1.Create new account

```
sudo adduser --home /home/username --shell /bin/bash --ingroup git username
```

###2.List all groups and the user names that were in each group

```
for u in `cut -f1 -d: /etc/passwd`; do echo -n $u:; groups $u; done | sort

groups $(cut -f1 -d":" /etc/passwd) | sort
```

###3.How to set up ssh so you aren't asked for a password

You can create a RSA authentication key to be able to log into a remote site from your account, without having to type your password.

Note that once you've set this up, if an intruder breaks into your account/site, they are given access to the site you are allowed in without a password, too! For this reason, this should never be done from root.

* Run ssh-keygen(1) on your machine, and just hit enter when asked for a password. 
This will generate both a private and a public key. With older SSH versions, they will be stored in ~/.ssh/identity and ~/.ssh/identity.pub; with newer ones, they will be stored in ~/.ssh/id_rsa and ~/.ssh/id_rsa.pub.
* Next, add the contents of the public key file into ~/.ssh/authorized_keys on the remote site (the file should be mode 600). 
If you are a developer and you want to access debian.org systems with such a key, it's possible to have the developer database propagate your key to all of the debian.org machines. See the LDAP gateway documentation.
You should then be able to use ssh to log in to the remote server without being asked for a password.

Reference:https://www.debian.org/devel/passwordlessssh

###4.Finding all files containing a text string on Linux

```
$ grep -rnw 'directory' -e "pattern"
// eg:
$ grep -rnw source/_posts/  -e "pragma"
```

Reference: http://stackoverflow.com/questions/16956810/finding-all-files-containing-a-text-string-on-linux

###5.Show bash script commands without executing them

There are two useful debug outputs for that task (both are written to stderr):

set -v mode (set -o verbose)
prints commands to be executed to stderr just like they are read from input (script file or keyboard)
prints everything before anything ( substitutions and expansions, …) big is applied
set -x mode (set -o xtrace)
prints everything like it really is executed, after substitutions and expansions applied
indicates the depth-level of the subshell (by default by preceeding a + (plus) sign to the shown command)
indicates the recognized words after word splitting by marking them like 'x y'
in a 4.1 version of the shell, this debug output can be printed to a configurable file descriptor (by setting the BASH_XTRACEFD variable) rather than stdout

Hint: These modes can be entered when calling Bash:

from commandline: bash -vx ./myscript
eventually (OS dependant) from shebang: #!/bin/bash -vx

Reference:http://wiki.bash-hackers.org/scripting/debuggingtips

###6.Toggle Web Sharing on or off in OSX 10.10

```
// Start
sudo apachectl start

// Stop
sudo apachectl stop

// Restart
sudo apachectl restart

// find the Apache version
httpd -v

// After starting Apache – test to see if the webserver is working in the browser – http://localhost – you should see the “It Works!” text.

```
Reference:http://coolestguidesontheplanet.com/get-apache-mysql-php-phpmyadmin-working-osx-10-10-yosemite/  

###7.List open ports on your machine (Mac OS X)
 
```
sudo lsof -i -P | grep -i "listen"
```
Reference:http://juretta.com/log/2007/08/08/list_open_ports_on_your_machine_mac_os_x_/  

###8.What do the numbers in a man page mean?

```
MANUAL SECTIONS
    The standard sections of the manual include:

    1      User Commands
    2      System Calls
    3      C Library Functions
    4      Devices and Special Files
    5      File Formats and Conventions
    6      Games et. Al.
    7      Miscellanea
    8      System Administration tools and Deamons

    Distributions customize the manual section to their specifics,
    which often include additional sections.
```

###9.jenkins

```
Note: When using launchctl the port will be 8080.

To have launchd start jenkins at login:
  ln -sfv /usr/local/opt/jenkins/*.plist ~/Library/LaunchAgents
Then to load jenkins now:
  launchctl load ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
Or, if you don't want/need launchctl, you can just run:
  jenkins
```
###10.Mac 上 Openssl 的配置文件路径
/System/Library/OpenSSL/openssl.cnf

###11.Find

find的使用实例：

搜索当前目录（含子目录，以下同）中，所有文件名以my开头的文件:

```
$ find . -name 'my*'
```

搜索当前目录中，所有文件名以my开头的文件，并显示它们的详细信息:

```
$ find . -name 'my*' -ls
```

搜索当前目录中，所有过去10分钟中更新过的普通文件。如果不加-type f参数，则搜索普通文件+特殊文件+目录:

```
$ find . -type f -mmin -10
```

搜索当前目录中，所有以my开头的目录：

```
$ find ~ -type d -name 'my*'
```

Reference:http://www.ruanyifeng.com/blog/2009/10/5_ways_to_search_for_files_using_the_terminal.html

###12.Enable Debian server pptpd's log and debug
To diagnose faults, enable the options debug dump in /etc/ppp/pptpd-options. The change is effective on the next connection. The debug output goes to /var/log/debug, and the dump output and usual output to /var/log/messages.

Referenc:http://poptop.sourceforge.net/dox/debian-howto.phtml

###13.iptables remove specific rules
Execute the same commands but replace the "-A" with "-D". For example:

```
$ iptables -A INPUT -i eth0 -p tcp --dport 443 -j ACCEPT
becomes

$ iptables -D INPUT -i eth0 -p tcp --dport 443 -j ACCEPT
```

###14.How to use tcpdump?
tcpdump can be used to capture the packets exchanged between the PPTP Client and the PPTP Server. By comparing the packets with what should be happening, you may determine what the cause of a problem might be.
Give to tcpdump the name of the network interface that connects to the PPTP Server, which for dial-up users would be ppp0, and for ADSL users eth0. Then, in another window or console, start the tunnel.

Start tcpdump in one window:	Then run pppd in the other window:

```
# tcpdump -i eth0
# pppd call tunnel
```

The tcpdump command should show you the packets as they are received or transmitted. Press Control/C when you do not need to capture any more packets.

You should see a connection to port 1723, followed by GRE packets in both directions. If you can see this, then you have full network connectivity. If you can't, you must find the problem.

Note: if you get an error:

tcpdump: pcap_loop: read: Network is down
then you may be using tcpdump on the wrong interface. Use ifconfig to list the available interfaces and choose the one through which your client must contact the server. See our diagrams if you're still confused.

The above technique is useful for quick checking, but for detailed analysis a binary tcpdump is required.

Security Warning
Internet addresses of your client and server, usernames and passwords are included in a binary tcpdump. This information may allow someone to gain equivalent access to protected networks. If you do not want to give away this information, convert it to text and remove it before sending the log to someone else.
If you have been asked to capture a complete tcpdump packet trace, for analysis by someone else, you should:

add -w file to tell it to save the packets to the file file,
add -s 0 to capture all of each packet,
and add tcp port 1723 or proto 47 to keep only the PPTP packets, if the client is performing other network traffic at the time.
For example:

```
# tcpdump -i eth0 -w my.tcpdump -s 0 tcp port 1723 or proto 47

```
Again, once the packets have been collected, use Control/C to stop the tcpdump process. The file containing the packets can then be e-mailed or analysed.

You may analyse it using ethereal.

```
# ethereal my.tcpdump
```
Note: when using ethereal, clicking on a byte in the hex dump will highlight the field name of the data in the packet, and vice versa. You may also analyse it using tcpdump:

```
# tcpdump -n -r my.tcpdump > my.tcpdump.txt
```
This converts it to text, saving the output into a file my.tcpdump.txt. This often hides the username and password. You may wish to globally substitute the IP addresses. Check your results to ensure your security.

###15.Viewing all iptables rules
You just need to specify the appropriate table:

```
$ iptables --table nat --list
$ iptables -t nat -L
```
If you don't specify a table, the filter table is used as the default. You can get even more information by including the -v or --verbose option.


