---
layout: post
title: "Linux命令行整理"
date: 2015-03-18 21:12:56 +0800
comments: true
categories: [Archives]
keywords: Linux Useful Command Line
discription: Linux Useful Command Line
---

###Create new account

```
sudo adduser --home /home/username --shell /bin/bash --ingroup git username
```

###List all groups and the user names that were in each group

```
for u in `cut -f1 -d: /etc/passwd`; do echo -n $u:; groups $u; done | sort

groups $(cut -f1 -d":" /etc/passwd) | sort
```

###How to set up ssh so you aren't asked for a password

You can create a RSA authentication key to be able to log into a remote site from your account, without having to type your password.

Note that once you've set this up, if an intruder breaks into your account/site, they are given access to the site you are allowed in without a password, too! For this reason, this should never be done from root.

* Run ssh-keygen(1) on your machine, and just hit enter when asked for a password. 
This will generate both a private and a public key. With older SSH versions, they will be stored in ~/.ssh/identity and ~/.ssh/identity.pub; with newer ones, they will be stored in ~/.ssh/id_rsa and ~/.ssh/id_rsa.pub.
* Next, add the contents of the public key file into ~/.ssh/authorized_keys on the remote site (the file should be mode 600). 
If you are a developer and you want to access debian.org systems with such a key, it's possible to have the developer database propagate your key to all of the debian.org machines. See the LDAP gateway documentation.
You should then be able to use ssh to log in to the remote server without being asked for a password.

Reference:https://www.debian.org/devel/passwordlessssh

###Finding all files containing a text string on Linux

```
grep -rnw 'directory' -e "pattern"
```

Reference: http://stackoverflow.com/questions/16956810/finding-all-files-containing-a-text-string-on-linux

###Show bash script commands without executing them

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

###Toggle Web Sharing on or off in OSX 10.10

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

###List open ports on your machine (Mac OS X)

```
sudo lsof -i -P | grep -i "listen"
```
Reference:http://juretta.com/log/2007/08/08/list_open_ports_on_your_machine_mac_os_x_/  