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