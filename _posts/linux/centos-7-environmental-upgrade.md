---
title: CentOS 7 环境升级
date: 2016-10-21 10:51:34
tags:
 - CentOS
category: 
 - Linux
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

docker 1.12 版本要求 CentOS 7 ，生产环境系统升级到 CentOS 7 后安装 docker 环境步骤简单记录备忘。

<!--more-->
### 升级 linux 内核
当前系统内核
```
# uname -r
3.10.0-327.el7.x86_64
```
**1. 下载新内核文件**

在 [https://cdn.kernel.org/](https://cdn.kernel.org/) 上看到当前最新稳定版本是 4.8.3 。
```
wget https://cdn.kernel.org/pub/linux/kernel/v4.x/linux-4.8.3.tar.xz
```


**2.解压**

拷贝到压缩包到 /usr/src 下
```
cp linux-4.8.3.tar.xz /usr/src
cd /usr/src
tar -xJf linux-4.8.3.tar.xz
```
**3. 清除残留编译文件**

```
cd linux-4.8.3
make mrproper && make clean
```
发现没有 gcc ，所以先安装 gcc 。
```
yum install -y gcc
```
安装完成后重新执行 make 。
**4. 生成编译配置表**

```
make oldconfig
```
默认配置，一路回车即可。
**5. 编译**

```
make
```
编译过程中报错 `openssl/opensslv.h：没有那个文件或目录`
```
yum install -y openssl-devel
```
安装完成后重新编译。
**6. 编译安装模块**

```
make modules && make modules_install
make install
```
**7. 修改启动项&重启**

到此安装结束，重启可以看到内核没变，有可能是启动顺序没改。
```
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg
CentOS Linux (4.8.3) 7 (Core)
CentOS Linux (3.10.0-327.el7.x86_64) 7 (Core)
CentOS Linux (0-rescue-22be0cd4be87429fa67b5df5203a9126) 7 (Core)
```
修改启动项
```
grub2-set-default 0
```
再重启就 OK 了。
```
uname -r
4.8.3
```
>如果对内核版本没有要求，只是更新到最新稳定版本，直接用 yum 简单 3 步即可完成。
>导入 Public Key
```
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-2.el7.elrepo.noarch.rpm
```
>升级内核
```
yum --enablerepo=elrepo-kernel install kernel-lt -y
```
>修改启动项
```
grub2-set-default 0
```

### 安装 docker
按照 docker 官网文档安装即可。
**1. 添加源**

```
sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
[dockerrepo]
name=Docker Repository
baseurl=https://yum.dockerproject.org/repo/main/centos/7/
enabled=1
gpgcheck=1
gpgkey=https://yum.dockerproject.org/gpg
EOF
```
**2. 安装**

```
yum install -y docker-engine
```
**3. 启动服务**

```
systemctl enable docker.service
```
**4. 启动镜像**

```
systemctl start docker
```
**5. 验证**

```
docker run --rm hello-world
```
**6. 创建 docker 组**

```
sudo groupadd docker
```
**7. 添加用户到 docker 组**

```
groupadd ops
useradd -G ops -g ops ops
sudo usermod -aG docker your_username`
```

### 设置 docker 的启动参数
docker 默认使用的存储启动是 device-mapper ,需要改成 overlay 。
**1. 安装 overlay**

```
sudo tee /etc/modules-load.d/overlay.conf <<-'EOF'
overlay
EOF
```
重启后确认是否安装成功。
```
lsmod | grep overlay
overlay                49152  0  
```
**2. 修改 docker 启动参数**

启用 docker.service 后，在 `/etc/systemd/system/multi-user.target.wants` 目录会生成 docker.service 文件连接，修改文件中的 `ExecStart=/usr/bin/dockerd` 。
```
cd /etc/systemd/system/multi-user.target.wants
```
添加启动参数
```
ExecStart=/usr/bin/dockerd --storage-driver=overlay2  --graph="/home/docker"
```
重启系统。


### 关闭 firewalld
firewalld 是 CentOS 7 引入的新服务，参考 [Firewalld 试用](../../../../2016/10/21/Firewalld-试用/)
```
systemctl stop firewalld.service
systemctl disable firewalld.service
```
### 关闭 selinux
```
vi /etc/selinux/config
SELINUX=disabled
```
### 文件共享

有 p75 和 p76 两台机器，需要将两台机器的 /u01/data/conf 目录共享。

**1. 安装**

```
yum install -y glusterfs glusterfs-server glusterfs-fuse
glusterfs --version
```
**2. 启动服务**

```
service glusterd start
service glusterfsd start
```
**3. 添加节点**

在任意节点执行即可。
```
gluster peer probe p76

gluster pool list
UUID					Hostname 	State
6a6962c3-42ab-49bf-9844-9e98cf419613	p76      	Connected
a0c173b4-341a-4f47-9eea-5e2ca9713665	localhost	Connected
```
**4. 创建数据目录**

```
mkdir -p /opt/data/conf
gluster volume create gv3 replica 2 transport tcp p75:/u01/data/conf p76:/u01/data/conf force
gluster volume info
```
**5. 启动**
```
gluster volume start gv3
```

**6. 安装 glusterfs 客户端**

将 p76 设置为客户端。

```
yum -y install glusterfs glusterfs-fuse
```
**7. 挂载目录**
```
mkdir -p /opt/data/conf
mount -t glusterfs -o rw p75:gv3 /opt/data/conf
```

至此，配置完成。在 p76 的 /opt/data/conf 目录放置的配置文件，在 p75 和 p76 的 /u01/data/conf 目录都能读取到。


>共享目录解决的是配置文件共享的问题，统一日志收集用ELK。