---
title: Ubuntu static IP
date: 2017-04-01 15:05:26
tags:
  - Ubuntu
category:
  - Linux
thumbnail: 
author: bsyonline
lede: 没有摘要
---

cat /etc/netplan/90-NM-14f59568-5076-387a-aef6-10adfcca2e26.yaml

```yaml
network:
  version: 2
  ethernets:
    ens33:
      renderer: NetworkManager
      match: {}
      addresses:
      - "192.168.93.211/24"
      nameservers:
        addresses:
        - 8.8.8.8
        - 114.114.114.114
      networkmanager:
        uuid: "14f59568-5076-387a-aef6-10adfcca2e26"
        name: "netplan-ens33"
        passthrough:
          connection.timestamp: "1726473685"
          ipv4.address1: "192.168.93.211/24,192.168.93.2"
          ipv4.method: "manual"
          ipv6.method: "disabled"
          ipv6.ip6-privacy: "-1"
          proxy._: ""
```


禁用防火墙和 selinux

```sh
sudo ufw disable
sudo setenforce 0 && sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```


安装 ssh server

```sh
sudo apt-get install -y openssh-server
sudo /etc/init.d/ssh start
```

root ssh

```sh
sudo vim /etc/ssh/sshd_config
#修改 PermitRootLogin 为 yes
sudo /etc/init.d/ssh restart
#修改root密码
sudo passwd
```

安装 docker

```sh
sudo apt update
sudo apt install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo usermod -aG docker $USER
docker --version
```

配置源和私服

```json
{
  "registry-mirrors": ["https://xxx.mirror.aliyuncs.com"],
  "dns": ["114.114.114.114", "8.8.8.8"],
  "insecure-registries": ["registry.k8s.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
```

重启 docker

```sh
systemctl daemon-reload
systemctl restart docker
```

配置 IPv4 和 iptables 参考 [Container Runtimes | Kubernetes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

```shell
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# sysctl params required by setup, params persist across reboots
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# Apply sysctl params without reboot
sudo sysctl --system
```

安装配置 Containerd

```sh
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo apt install containerd.io -y

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml
sudo systemctl restart containerd
```









安装 cri-dockerd

在[release](https://github.com/Mirantis/cri-dockerd/releases)下载构建好的对应Ubuntu版本的`.deb`安装文件。

```sh 
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.15/cri-dockerd_0.3.15.3-0.ubuntu-jammy_amd64.deb
dpkg -i cri-dockerd_0.3.15.3-0.ubuntu-jammy_amd64.deb
systemctl restart cri-docker.service
```









安装k8s

添加源

```sh
apt update && apt install -y apt-transport-https

#添加公钥
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 

#添加源
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF

#更新缓存索引
apt update

#查看kubectl可用版本
apt-cache madison kubectl

#指定版本安装
apt install -y kubectl=1.25.3-00 kubelet=1.25.3-00 kubeadm=1.25.3-00

#开机启动
systemctl enable kubelet
```

master节点

```sh
kubeadm init \
--kubernetes-version=1.25.3 \
--apiserver-advertise-address=192.168.93.211 \
--image-repository=registry.k8s.com/google_containers \
--pod-network-cidr="10.244.0.0/16" \
--service-cidr="10.96.0.0/12" \
--ignore-preflight-errors=Swap \
--cri-socket=unix:///var/run/cri-dockerd.sock
--cri-socket=unix:///var/run/containerd/containerd.sock
```