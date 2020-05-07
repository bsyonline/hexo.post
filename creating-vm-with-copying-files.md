---
title: 拷贝 vmware 文件方式创建虚拟机
date: 2015-08-29 15:53:17
tags:
 - VMware
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

### vmware打开文件

### 查看ifconfig -a发现没有eth0网卡
#### 打开 /etc/udev/rules.d/70-persistent-net.rules文件，将eth1改成eth0。  
#### 将/etc/sysconfig/network-scripts/ifcfg-eth0中的HWADDR改成当前虚拟机的mac地址。

### reboot
