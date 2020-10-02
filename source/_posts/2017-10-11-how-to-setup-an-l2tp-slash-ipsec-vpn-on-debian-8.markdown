---
layout: post
title: "How to setup a IKEv2 VPN Server on CentOS 8"
date: 2017-10-11 08:32:44 +0800
comments: true
categories: [Archives]
keywords: strongswan, eap-mschapv2, VPN, IKEv2, CentOS 8, iPhone, iOS, macOS, windows 10
description: How to setup a IKEv2 VPN Server on CentOS 8
---

由于 Debian 8 官方不再支持，经过一番考虑和准备，最近将 VPS 的系统换成了 CentOS 8。Libreswan 的文档没有 VPN 时打不开，这就形成了一个死循环，所以将她换成了 StrongSwan 。StrongSwan 的文档个人觉得对新手不是很友好，我是看了好一会才看明白些，另外在支持 macOS，iOS，Windows 10 的过程中也遇到不少问题，所以这里记录下来，以备日后参考。  

[StrongSwan 的文档入口](https://www.strongswan.org/documentation.html)页有[安装指南](https://wiki.strongswan.org/projects/strongswan/wiki/InstallationDocumentation)和[可用的配置示例](https://wiki.strongswan.org/projects/strongswan/wiki/UsableExamples)。我个人建议先看下 [Introduction to strongSwan](https://wiki.strongswan.org/projects/strongswan/wiki/IntroductionTostrongSwan)，这对安装和配置很有帮助。  

CentOS 8 中的 epel 仓库中有预编译的 strongswan 二进制安装包，我们直接安装就可以了。接下来就是配置，strongswan 有四种认证方式，出于安全和便捷的考虑，我选择了 Extensible Authentication Protocol (EAP)。  

首先是准备证书。我们可以使用自签名证书或者通用证书颁发机构签发的证书，后者不需要在各个终端上安装自签名根证书，所以部署上更简单。  

我们先来看看下自签名证书。  

生成根证书:

{% codeblock %}
cd /etc/strongswan/swanctl
strongswan pki --gen --outform pem > rsa/ca.pem
strongswan pki --self --ca --in rsa/ca.pem --type rsa --dn "C=CH, O=strongSwan, CN=strongSwan Root CA" --outform pem > x509ca/ca-cert.pem

{% endcodeblock %}

生成服务器证书:  

{% codeblock %}
cd /etc/strongswan/swanctl
strongswan pki --gen --outform pem > rsa/server.pem
strongswan pki --pub --in rsa/server.pem --type rsa | strongswan pki --issue --cacert cacerts/ca.cert.pem --cakey rsa/ca.pem --dn "C=CH, O=strongSwan, CN=swan.tenneshop.com" --san swan.tenneshop.com --flag serverAuth --flag ikeIntermediate --outform pem > x509/server-cert.pem

{% endcodeblock %}
<!--more-->
再来看下使用 Let's Encrypt 签发的证书。先按照官方指南安装并获取证书，然后安装到 StrongSwan：

{% codeblock %}
# 根证书
cd /etc/strongswan/swanctl/x509ca
ln -s /etc/letsencrypt/live/www.example.com/chain.pem lets-encrypt-authority-x3-ca.pem

# 服务器证书
cd /etc/strongswan/swanctl/x509
ln -s /etc/letsencrypt/live/www.example.com/cert.pem www-example-com.pem

# 服务器证书私钥
cd /etc/strongswan/swanctl/private/
ln -s /etc/letsencrypt/live/www.example.com/privkey.pem www-example-com-privkey.pem
{% endcodeblock %}

这里值得一提的是我们需要将 Let's Encrypt 这个中间证书安装到 `/etc/strongswan/swanctl/x509ca` 目录下，不然 Windows 10 连接不成功。

两种方式我都折腾成功，因为现在 Let's Encrypt 的免费证书很方便，考虑到便捷，最终我使用了通用证书颁发机构签发的证书。因为暂时没有找到关闭 ipsec 配置的方法，我将 ipsec.d 目录重命令为 ipsec-off.d 来 workaround。

其次是具体配置。新版本的 StrongSwan 推荐使用 swanctl.conf 来配置，而官方文档上 swanctl.conf 和 ipsec.conf 的配置信息杂糅在一起，网络上最大多是 ipsec.conf 的配置，从面向未来的角度考虑，我是完全使用 swanctl.conf ，这也让我有种神清气爽的感觉。我这张配置信息经过脱敏后如下：  

{% codeblock %}
# Section defining IKE connection configurations.
connections {

    	ikev2-eap-mschapv2 {
        	version = 2
        	proposals = aes256-sha1-modp1024,aes128-sha1-modp1024,3des-sha1-modp1024
        	rekey_time = 0s
        	pools = primary-pool-ipv4, primary-pool-ipv6
        	fragmentation = yes
        	dpd_delay = 30s
		send_cert = always
        	# dpd_timeout doesn't do anything for IKEv2. The general IKEv2 packet timeouts are used.
        	local-1 {
            		certs = www-example-com.pem
            		id = @www.example.com
        	}
        	remote-1 {
            		auth = eap-mschapv2
            		# go ask the client for its eap identity.
            		eap_id = %any
        	}
        	children {
            		ikev2-eap-mschapv2 {
                		local_ts = 0.0.0.0/0,::/0
                		rekey_time = 0s
                		dpd_action = clear
				esp_proposals = aes256-sha1,aes256-sha1,3des-sha1
            		}
        	}
    	}
}

secrets {
	
	private-www {
		file = www-example-com-privkey.pem
	}

	eap-carol {
        id = carol
        secret = "carolspassword" 
   }

}

# Section defining named pools.
pools {
    primary-pool-ipv4 {
        addrs = 10.11.1.0/24
        dns = 8.8.8.8,8.8.4.4
    }
    primary-pool-ipv6 {
        addrs = fec1::0/16

    }
}

# Include config snippets
include conf.d/*.conf
{% endcodeblock %}

再次是配置转发和防火墙。 

开启 IPv4 转发  

{% codeblock %}
echo 1 > /proc/sys/net/ipv4/ip_forward
echo 0 > /proc/sys/net/ipv4/conf/all/accept_redirects
echo 0 > /proc/sys/net/ipv4/conf/all/send_redirects
{% endcodeblock %}

这个是临时性的，如果想永久更改，则修改 `/etc/sysctl.conf ` 

{% codeblock %}
net.ipv4.ip_forward = 1
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.send_redirects = 0
{% endcodeblock %}

允许 IPSec 端口监听  

{% codeblock %}
iptables -A INPUT -p udp --dport 500 --j ACCEPT
iptables -A INPUT -p udp --dport 4500 --j ACCEPT
iptables -A INPUT -p esp -j ACCEPT
{% endcodeblock %}

允许 VPN 到外网的流量  

{% codeblock %}
iptables -t nat -A POSTROUTING -s 10.11.1.0/24 -o eth0 -j MASQUERADE 
{% endcodeblock %}

防火墙配置可以写到系统防火墙配置文件或利用开机启动脚本设置。我是参考鸟哥的做法，使用开机启动脚本设置：  

防火墙设置脚本：  

{% codeblock %}
# /usr/local/virus/iptables/iptables.rule
#!/bin/bash

# Network card infos
EXTIF="eth0"
INIF="eth0"
INNET="10.11.1.0/24"

export EXTIF INIF INNET

# Reset rules
PATH=/sbin:/usr/sbin:/bin:/usr/bin:/usr/local/sbin:/usr/local/bin
export PATH

iptables -F
iptables -X
iptables -Z
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

# Allowed services
iptables -A INPUT -p tcp -i $EXTIF --dport 21 --sport 1024:65534 -j ACCEPT #FTP
iptables -A INPUT -p tcp -i $EXTIF --dport 22 --sport 1024:65534 -j ACCEPT #SSH
iptables -A INPUT -p tcp -i $EXTIF --dport 80 --sport 1024:65534 -j ACCEPT #HTTP
iptables -A INPUT -p tcp -i $EXTIF --dport 8080 --sport 1024:65534 -j ACCEPT #HTTP
iptables -A INPUT -p tcp -i $EXTIF --dport 443 --sport 1024:65534 -j ACCEPT #HTTPS
iptables -A INPUT -p tcp -i $EXTIF --dport 8443 --sport 1024:65534 -j ACCEPT #HTTPS
# Frp
iptables -A INPUT -p tcp -i $EXTIF --dport 8008 --sport 1024:65534 -j ACCEPT
iptables -A INPUT -p tcp -i $EXTIF --dport 48557 --sport 1024:65534 -j ACCEPT
iptables -A INPUT -p tcp -i $EXTIF --dport 9090 --sport 1024:65534 -j ACCEPT
iptables -A INPUT -p tcp -i $EXTIF --dport 48559 --sport 1024:65534 -j ACCEPT

# Reset nat rules

iptables -F -t nat
iptables -X -t nat
iptables -Z -t nat
iptables -t nat -P PREROUTING ACCEPT
iptables -t nat -P POSTROUTING ACCEPT
iptables -t nat -P OUTPUT ACCEPT

# Setup ip share if need
if [ "$INIF" != "" ]; then

	iptables -A INPUT -i $INIF -j ACCEPT

	echo "1" > /proc/sys/net/ipv4/ip_forward
	if [ "$INNET" != "" ]; then
		for innet in $INNET
		do
			iptables -t nat -A POSTROUTING -s $innet -o $EXTIF -j MASQUERADE
			# Redirect 80 to 8080, 443 to 8443 unless it from innet
			iptables -t nat -A PREROUTING -j REDIRECT -p tcp ! -s $innet --destination-port 80 --to-ports 8080
			iptables -t nat -A PREROUTING -j REDIRECT -p tcp ! -s $innet --destination-port 443 --to-ports 8443
		done
	fi

fi
{% endcodeblock %}

开机启动脚本： 

{% codeblock %}
/etc/systemd/system/custom-iptables.service
[Unit]
Description=Custom firewall

[Service]
Type=oneshot
ExecStart=/bin/bash /usr/local/virus/iptables/iptables.rule

[Install]
WantedBy=multi-user.target
{% endcodeblock %}

开启开机启动：`systemctl enable custom-iptables.service`

最后就是各终端连接。IKEv2 的连接很容易， macOS，iOS，Windows 10 都内置支持。  

macOS 的具体连接为：System Preferences > Network > + > Interface: VPN, VPN Type: IKEv2, Service Name: your favorite service name ，最后填上服务器域名，Remote ID 和用户的帐号密码就能连接了，Remote ID 通常就是服务器域名。iOS 和 Windows 10 的连接和 macOS 类似。

###问题解决

安装配置过程中可能会遇到各种各样的问题，这时候我们首先是要开启 StrongSwan 的调试日志，这样会输出更多信息帮助我们解决问题。  

{% codeblock %}
#/etc/strongswan/strongswan.d/charon-logging.conf

charon {

    # Section to define file loggers, see LOGGER CONFIGURATION in
    # strongswan.conf(5).
    filelog {

        # <name> may be the full path to the log file if it only contains
        # characters permitted in section names. Is ignored if path is
        # specified.
        # <name> {

            # Loglevel for a specific subsystem.
            # <subsystem> = <default>

            # If this option is enabled log entries are appended to the existing
            # file.
            # append = yes

            # Default loglevel.
            # default = 1

            # Enabling this option disables block buffering and enables line
            # buffering.
            # flush_line = no

            # Prefix each log entry with the connection name and a unique
            # numerical identifier for each IKE_SA.
            # ike_name = no

            # Optional path to the log file. Overrides the section name. Must be
            # used if the path contains characters that aren't allowed in
            # section names.
            # path =

            # Adds the milliseconds within the current second after the
            # timestamp (separated by a dot, so time_format should end with %S
            # or %T).
            # time_add_ms = no

            # Prefix each log entry with a timestamp. The option accepts a
            # format string as passed to strftime(3).
            # time_format =

        # }

	charon-debug-log {
                    # this setting is required with 5.7.0 and newer if the path contains dots
                    path = /var/log/charon_debug.log
                    time_format = %a, %Y-%m-%d, %H:%M:%S
                    default = 2
                    mgr = 0
                    net = 1
                    enc = 1
                    asn = 1
                    job = 1
                    ike_name = yes
                    append = no
                    flush_line = yes
            }

    }

    # Section to define syslog loggers, see LOGGER CONFIGURATION in
    # strongswan.conf(5).
    syslog {

        # Identifier for use with openlog(3).
        # identifier =

        # <facility> is one of the supported syslog facilities, see LOGGER
        # CONFIGURATION in strongswan.conf(5).
        # <facility> {

            # Loglevel for a specific subsystem.
            # <subsystem> = <default>

            # Default loglevel.
            # default = 1

            # Prefix each log entry with the connection name and a unique
            # numerical identifier for each IKE_SA.
            # ike_name = no

        # }

    }

}
{% endcodeblock %}

既然我们已经将日志输出到专门的文件了我们可能不想让她继续输出到系统日志文件中，这样我可以减轻在系统日志排查其他问题时的干扰信息，具体设置是在 `/etc/strongswan/strongswan.d/charon-systemd.conf` :  

{% codeblock %}
charon-systemd {

    # Section to configure native systemd journal logger, very similar to the
    # syslog logger as described in LOGGER CONFIGURATION in strongswan.conf(5).
    journal {

        # Loglevel for a specific subsystem.
        # <subsystem> = <default>

        # Default loglevel.
        default = -1

    }

}
{% endcodeblock %}

其次是 swanctl 命令可以动态加载连接和密钥配置，以及查看加载的证书，是个不错的帮手。  

{% codeblock %}
swanctl --load-conns
swanctl --load-creds
swanctl --list-certs
{% endcodeblock %}

有时日志也没输出什么有效信息，我们只能用有限的线索求助 Google 了。  

##Reference:
 
* [Introduction to strongSwan](https://wiki.strongswan.org/projects/strongswan/wiki/IntroductionTostrongSwan)  
* [Usable Examples configurations](https://wiki.strongswan.org/projects/strongswan/wiki/UsableExamples)  
* [Requesting Help and Reporting Bugs](https://wiki.strongswan.org/projects/strongswan/wiki/HelpRequests)  
* [Migration from ipsec.conf to swanctl.conf](https://wiki.strongswan.org/projects/strongswan/wiki/Fromipsecconf)  
* [iOS (Apple iPhone, iPad...) and macOS](https://wiki.strongswan.org/projects/strongswan/wiki/AppleClients)  
* [ipsec](https://wiki.strongswan.org/projects/strongswan/wiki/IpsecCommand)  
* [Issue #772](https://wiki.strongswan.org/issues/772)  
* [Debian + StrongSwan 配置 IKEv2 VPN 科学上网](https://ericfu.me/debian-strongswan-ikev2-vpn/)  
* [折腾搬瓦工–09–为iPhone配置证书认证的VPN](https://wbuntu.com/p/499/)  
* [Strongswan with letsencrypt certificates (IKEv2-EAP)](https://serverfault.com/questions/956674/strongswan-with-letsencrypt-certificates-ikev2-eap)   
* [Setting-up a Simple CA Using the strongSwan PKI Tool](https://wiki.strongswan.org/projects/strongswan/wiki/SimpleCA)  
* [charon-systemd](https://wiki.strongswan.org/projects/strongswan/wiki/Charon-systemd)  

##修改记录  

* 2020/10/02: 记录关闭输出到系统日志文件的配置
* 2020/09/07: 修改成 CentOS 8 上安装 StrongSwan
* 2017/10/11: 第一次完成


