---
title: ubuntu 24.04 installation
date: 2017-04-01 15:05:26
tags:
  - Ubuntu
category:
  - Linux
thumbnail: 
author: bsyonline
lede: 没有摘要
---

### 设置静态IP

live server 安装网络 IPv4 Method Manual 配置：

| 配置项            | 值                       |
| -------------- | ----------------------- |
| Subnet         | 192.168.93.0/24         |
| Address        | 192.168.93.212          |
| Gateway        | 192.168.93.2            |
| Name Server    | 192.168.93.2            |
| Search domains | 8.8.8.8,114.114.114.114 |
配置完 /etc/netplan/50-cloud-init.yaml 如下

```yaml
network:
    ethernets:
        ens33:
            addresses:
            - 192.168.93.212/24
            nameservers:
                addresses:
                - 192.168.93.2
                search:
                - 8.8.8.8
                - 114.114.114.114
            routes:
            -   to: default
                via: 192.168.93.2
    version: 2
```

### 安装配置 ssh server

安装时选了就不用装了

```sh
sudo apt install -y openssh-server
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


### 配置 Container Runtimes
参考 [Container Runtimes | Kubernetes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

配置内核和 IPv4 packet forwarding

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


安装 Containerd

```sh
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates

sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/containerd.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update && sudo apt install containerd.io -y

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

#修改 sandbox_image 的值为国内镜像或私有镜像。
sudo sed -i 's/registry.k8s.io/reg.k8s.me/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl status containerd
containerd -v
```


/etc/crictl.yaml

```sh
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
```

/etc/containerd/config.toml

```toml
	...
	
    [plugins."io.containerd.grpc.v1.cri".registry]
      config_path = ""

      [plugins."io.containerd.grpc.v1.cri".registry.auths]

      [plugins."io.containerd.grpc.v1.cri".registry.configs]
        [plugins."io.containerd.grpc.v1.cri".registry.configs."reg.k8s.me".tls]
          insecure_skip_verify = true
        [plugins."io.containerd.grpc.v1.cri".registry.configs."reg.k8s.me"]
          [plugins."io.containerd.grpc.v1.cri".registry.configs."reg.k8s.me".auth]
            username = "admin"
            password = "123456"


      [plugins."io.containerd.grpc.v1.cri".registry.headers]

      [plugins."io.containerd.grpc.v1.cri".registry.mirrors]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."docker.io"]
          endpoint = ["https://xxx.mirror.aliyuncs.com"]
        [plugins."io.containerd.grpc.v1.cri".registry.mirrors."reg.k8s.me"]
	        endpoint = ["http://reg.k8s.me"]

	...

```

配置完成后重启 containerd

```sh
sudo systemctl restart containerd
crictl pull reg.k8s.me/library/alpine:latest
```




```sh
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.25/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.25/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list


sudo apt update
sudo apt install -y kubelet kubeadm kubectl

```

### 禁用 swap

```sh
sudo swapoff -a
sudo sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab
sudo swapon --show
```

修改 /etc/sysconfig/kubelet

```
KUBELET_EXTRA_ARGS="--fail-swap-on=false"
```

### 禁用防火墙和 selinux

```sh
sudo ufw disable
sudo apt install -y selinux-utils
sudo setenforce 0 && sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```

### 拉取镜像

```sh
sudo kubeadm config images pull \
--image-repository=registry.aliyuncs.com/google_containers \
--kubernetes-version=v1.25.16 \
--cri-socket=unix:///run/containerd/containerd.sock


images=$(kubeadm config images list --kubernetes-version=1.25.16 | awk -F "/" '{print $NF}')
for i in ${images} 
do   
  docker pull registry.aliyuncs.com/google_containers/$i
  docker tag registry.aliyuncs.com/google_containers/$i reg.k8s.me/google_containers/$i
  docker push reg.k8s.me/google_containers/$i
  docker rmi registry.aliyuncs.com/google_containers/$i
done

```

### 初始化

```bash
sudo kubeadm init \
--apiserver-advertise-address=192.168.93.201 \
--image-repository=reg.k8s.me/google_containers \
--kubernetes-version=v1.25.16 \
--service-cidr=10.96.0.0/12 \
--pod-network-cidr=10.244.0.0/16 \
--cri-socket=unix:///run/containerd/containerd.sock
```

master 初始化成功后按提示添加 node 到集群。

```sh
kubeadm join 192.168.93.201:6443 --token 5xqjs2.plllmvhmynr493z6 --discovery-token-ca-cert-hash sha256:a23f62ed6fb9dfedf7dceaae295bb3cd90aa1ef7f21b89a4ef24108e3fed0b0b
```

### 安装 calico


```sh
wget https://raw.githubusercontent.com/projectcalico/calico/v3.28.2/manifests/calico.yaml

sed -i 's/docker.io/reg.k8s.me/g' calico.yaml

kubectl apply calico.yaml

```



### 自动补全命令

```shell
echo "source <(kubectl completion bash)" >> ~/.bashrc 
echo "source <(kubeadm completion bash)" >> ~/.bashrc 
source ~/.bashrc
```


