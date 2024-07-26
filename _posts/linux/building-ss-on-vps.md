---
title: Building ShadowSocks on VPS
tags:
  - ShadowSocks
category:
  - Wiki
author: bsyonline
lede: 没有摘要
date: 2018-12-13 18:36:06
thumbnail:
---



Shadowsocks 一种基于 Socks5 代理方式的加密传输协议，也可以指实现这个协议的各种开发包。Shadowsocks 分为服务器端和客户端。VPS（Virtual Private Server 虚拟专用服务器）是指将一台服务器分割成多个虚拟专享服务器提供服务的技术，我们这里的 VPS 是指 VPS 主机。本篇我们就来介绍一下如何将 Shadowsocks 服务端安装在 VPS 主机上。VPS 提供商有很多，这里以阿里云 ECS 主机为例。

#### Shadowsock Server 配置

安装依赖包

```
# yum install wget curl curl-devel zlib-devel openssl-devel perl perl-devel cpio expat-devel gettext-devel git autoconf libtool gcc swig python-devel lsof
```

下载 setuptools

```
# cd /usr/local/src/
# wget --no-check-certificate  https://pypi.python.org/packages/source/s/setuptools/setuptools-19.6.tar.gz#md5=c607dd118eae682c44ed146367a17e26
# tar -zvxf setuptools-19.6.tar.gz
# cd setuptools-19.6
# python2.7 setup.py build
```

安装 pip

```
# yum -y install epel-release
# yum -y install pip python-pip
# pip install --upgrade pip
```

安装 shadowsocks

```
# pip install shadowsocks
```

安装加密的依赖包

```
# pip install M2Crypto
```

创建服务端配置文件 /etc/shadowsocks.json

```
{
"server": "0.0.0.0",
"server_port": 8388,
"password": "your ss password",
"timeout": 300,
"method": "aes-256-cfb"
}
```

创建 ss 服务并启动

修改 /etc/systemd/system/ssserver.service

```
[Unit]
Description=ssserver
[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json
[Install]
WantedBy=multi-user.target
```

启动服务

```
# systemctl enable ssserver
# systemctl start ssserver
```

检查服务状态

```
# systemctl status ssserver -l
● ssserver.service - ssserver
   Loaded: loaded (/etc/systemd/system/ssserver.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2018-12-14 16:07:34 CST; 8s ago
 Main PID: 1338 (ssserver)
   CGroup: /system.slice/ssserver.service
           └─1338 /usr/bin/python2 /usr/bin/ssserver -c /etc/shadowsocks.json

Dec 14 16:07:34 izj6cgrp5ibxaav9i9x2t6z systemd[1]: Started ssserver.
Dec 14 16:07:34 izj6cgrp5ibxaav9i9x2t6z systemd[1]: Starting ssserver...
Dec 14 16:07:34 izj6cgrp5ibxaav9i9x2t6z ssserver[1338]: INFO: loading config from /etc/shadowsocks.json
Dec 14 16:07:34 izj6cgrp5ibxaav9i9x2t6z ssserver[1338]: 2018-12-14 16:07:34 INFO     loading libcrypto from libcrypto.so.10
Dec 14 16:07:34 izj6cgrp5ibxaav9i9x2t6z ssserver[1338]: 2018-12-14 16:07:34 INFO     starting server at 0.0.0.0:8388
```

```
# lsof -i:8388
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
ssserver 1444 root    3u  IPv4  17138      0t0  TCP *:8388 (LISTEN)
ssserver 1444 root    4u  IPv4  17139      0t0  UDP *:8388 
```

在配置好 ECS 主机之后，远程 telnet 主机的 8388 会发现无法连接，可能的原因是被防火墙拦截了，所以我们需要把防火墙关掉。

```
# systemctl stop firewalld.service
# systemctl disable firewalld.service
# firewall-cmd --state
not running
```

还要把 8388 端口开放出来。可以使用 iptables 来管理开放端口，首先需要安装 iptables 。
```
# yum install iptables-services
# systemctl restart iptables.service
# systemctl enable iptables.service
```

添加配置
```
# iptables -A INPUT -p tcp --dport 8388 -j ACCEPT
# iptables -A OUTPUT -p tcp --sport 8388 -j ACCEPT
```

保存配置

```
# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
```

查看配置
```
# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            state RELATED,ESTABLISHED
ACCEPT     icmp --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:22
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            state NEW tcp dpt:8388
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:8388

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         
REJECT     all  --  0.0.0.0/0            0.0.0.0/0            reject-with icmp-host-prohibited

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         
ACCEPT     tcp  --  0.0.0.0/0            0.0.0.0/0            tcp spt:8388
```

如果防火墙已经关闭，端口也已经开放还是无法访问，就需要开启阿里云端口限制。

![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181213/233904777.png)

![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181213/233557059.png)

<img src="https://raw.githubusercontent.com/bsyonline/pic/master/20181213/233442475.png" style="width: 350px"/>

规则设置好后 telnet 一下，访问正常 Shadowsock Server 就配置好了。

### 客户端配置
windows 上的配置比较简单，下载 Shadowsocks 客户端，配置服务器信息即可。
<img src="https://raw.githubusercontent.com/bsyonline/pic/master/20181213/235222900.png" style="width: 350px"/>
ubuntu 上也有类似的客户端 shadowsock-qt5 ，安装也很简单。
```
# sudo add-apt-repository ppa:hzwhuang/ss-qt5
# sudo apt-get update
# sudo apt-get install shadowsocks-qt5
``` 

配置好客户端还需要配合浏览器插件，firefox 和 chrome 都可以使用 SwitchyOmega 。

![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181213/20181214135728.png)
![mark](https://raw.githubusercontent.com/bsyonline/pic/master/20181213/20181214140036.png)

