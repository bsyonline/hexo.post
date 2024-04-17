---
title: k8s
tags:
  - cloud native
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2018-05-31 23:20:09
thumbnail:
---



### 一. 介绍

从 2012 年开始，在虚拟化领域 docker 成为一种广泛使用的技术。docker 就像一个轻量级的虚拟机，它将程序代码和环境配置封装到一个容器中，我们可以通过 docker run 启动容器。随着服务增多，服务之间的关系变得复杂，就有了 docker-compose 。除了维护复杂的关系，我们还希望根据访问量自动分配服务器、网络等资源，宕机的时候还能够自动恢复，以及在多台服务器上管理这些容器，docker-compose 就不方便了，kubernetes 就应运而生。kubernetes 简称 k8s ，是一个用于容器应用的管理，部署和扩展的开源管理平台。它管理着一系列的主机或服务器作为 Node 节点，每个 Node 节点运行着若干个相互独立的 Pod ，Pod 是一个或多个容器的集合，是 kubernetes 中可部署的最小执行单元。kubernetes 通过一个中心控制器来管理和协调 Node 节点，这个中心控制器就是 Control Plane ，他们之间通过 API 进行通信，实时监测节点的网络状态来平衡服务器资源，还可以通过下发指令处理任务，比如某个 pod 挂掉了，kubernetes 就从提前准备好的备份容器中启用一个来替换 pod ，这些备份容器叫做 Replica Set 。Node 和 Control Plane 组成了 kubernetes 集群。

### 二. 概念术语

[Glossary | Kubernetes](https://kubernetes.io/docs/reference/glossary/?fundamental=true)

1. spec：期望状态，在 yaml 文件中描述。
2. status：当前状态，由 Kubernetes 系统和组件设置并更新。k8s 会尽可能让 status 和 spec 相匹配。
3. control-plane：管理集群的多个组件的集合，就是 master 。
4. kube-apiserver：是 control-plane 组件，负责处理接收请求。
5. kube-scheduler：是 control-plane 组件，负责监视 Pod， 并选择 Node 来让 Pod 在上面运行。
6. kube-controller-manager：是 control-plane 的组件， 负责运行 controller 进程。
7. cloud-controller-manager：是 control-plane 的组件， 负责对接云提供商的 API 。
8. etcd：control-plane 用来存储 Kubernetes 所有集群数据。
9. Master：管理节点，等同于 control-plane 。
10. Node：工作节点，负责维护运行的 Pod 并提供 Kubernetes 运行环境。
11. kubectl：Kubernetes API 与 control-plane 进行通信的命令行工具。会在集群中每个 Node 上运行。
12. kube-proxy：是集群中每个 Node 上所运行的网络代理，负责 Pod 与集群内外的网络通信。
13. Container Runtime：负责运行容器的软件，可以有多种实现，如 docker，containerd，CRI-O 等。
14. Minikube：本地运行 Kubernetes 的一种工具。
15. kubeadm：用来快速安装 Kubernetes 并搭建安全稳定的集群的工具。
16. Pod：可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。
17. Label：标签是一些关联到 Pods 这类对象上的键值对，用来为对象设置可标识的属性标记，通常用来组织和选择对象子集。
18. PodTemplate: Pod 的定义的模板。
19. ReplicationController：通常缩写为 "rc"，用来管理控制 pod 数量。
20. ReplicaSet：通常缩写为 "rs"，由 rc 发展而来，比 rc 功能更多。
21. deployment：ReplicaSet 的替代方案。deployment 是一个可以拥有 ReplicaSet 并使用声明式方式在服务器端完成对 Pod 滚动更新的对象。
22. service：可以让一个或一组 Pod 在网络上被访问到。
23. endpoint：实际服务的端点，ip + port 。
24. ingress：允许入站连接到达后端定义的端点的规则集合，可以配置为向服务提供外部可访问的 URL、负载均衡流量、终止 SSL、提供基于名称的虚拟主机等。
25. namespace：一种隔离机制，可以将同一集群中的资源划分为相互隔离的组。
26. HPA：弹性伸缩，Pod 水平自动扩缩器（Horizontal Pod Autoscaler）是一种 API 资源，它根据目标 CPU 利用率或自定义度量目标扩缩 Pod 副本的数量。
27. LimitRange：提供约束来限制命名空间中每个 Container 或 Pod 的资源消耗。
28. ClusterRole：
29. ClusterRoleBinding：集群级别的角色绑定。
30. PV
31. PVC
32. SC
33. 
34. 
35. 
36. 
37. 

### 三. 环境安装

kebernetes 集群部署有几种方式：

1. 本地/单机部署：Kind、minikube
2. **分布式集群部署：利用 kubeadm**
3. 二进制部署

#### 准备 centos 7 环境

配置 host

```
192.168.93.201 k8s-master.com k8s-master
192.168.93.202 k8s-node1.com  k8s-node1
192.168.93.203 k8s-node2.com  k8s-node2
192.168.93.204 registry.k8s.com  k8s-registry
```

配置主机名

```
hostnamectl set-hostname k8s-master
```

配置静态ip

```
# cat /etc/sysconfig/network-scripts/ifcfg-eth0
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=58fd05ce-43bf-4bd8-9ef3-2888746afea9
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.93.201
PREFIX=24
GATEWAY=192.168.93.2
DNS1=114.114.114.114
```

关闭防火墙

```
systemctl disable firewalld.service
systemctl stop firewalld.service
```

关闭SELinux

```
sudo setenforce 0
sudo sed -i 's/^SELINUX=enforcing$/SELINUX=disabled/' /etc/selinux/config
```

关闭swap

```
sed -i '/swap/s/^\(.*\)$/#\1/g' /etc/fstab
```

更新yum源

```
sed -e 's|^mirrorlist=|#mirrorlist=|g' \
    -e 's|^#baseurl=http://mirror.centos.org/centos|baseurl=https://mirrors.tuna.tsinghua.edu.cn/centos|g' \
    -i.bak \
    /etc/yum.repos.d/CentOS-*.repo
    
yum makecache
```



安装工具

```
yum install -y net-tools wget vim* lrzsz bash-completion
```



#### 安装 docker engine

从 [阿里巴巴开源镜像站-OPSX镜像站-阿里云开发者社区 (aliyun.com)](https://developer.aliyun.com/mirror/)  获取 docker-ce 。

```
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
# Step 4: 更新并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 5: 开启Docker服务
sudo systemctl enable docker.service
sudo systemctl start docker
```



#### 安装 CRI-O

##### 配置 IPv4 和 iptables 

参考 [Container Runtimes | Kubernetes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)

```
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



##### 配置 cgroup driver

```
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "registry-mirrors": ["https://xqnrfb7c.mirror.aliyuncs.com"],
  "dns": ["114.114.114.114", "8.8.8.8"],
  "insecure-registries": ["registry.k8s.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

# 重启 docker
sudo systemctl restart docker
```



##### 安装 cri-dockerd

```
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.2/cri-dockerd-0.3.2.amd64.tgz
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.service
wget https://raw.githubusercontent.com/Mirantis/cri-dockerd/master/packaging/systemd/cri-docker.socket
tar -zxf cri-dockerd-0.3.2.amd64.tgz

cp cri-dockerd/cri-dockerd /usr/local/bin
cp cri-docker.service /etc/systemd/system/
cp cri-docker.socket /etc/systemd/system/
```

把 cri-dockerd 命令 scp 到其他机器的 /usr/local/bin 和 /usr/bin 。

修改配置

```
# cri-dockerd.service
cat <<EOF | sudo tee /usr/lib/systemd/system/cri-docker.service
[Unit]
Description=CRI Interface for Docker Application Container Engine
Documentation=https://docs.mirantis.com
After=network-online.target firewalld.service docker.service
Wants=network-online.target
Requires=cri-docker.socket
[Service]
Type=notify
ExecStart=/usr/local/bin/cri-dockerd --network-plugin=cni --pod-infra-container-image=registry.aliyuncs.com/google_containers/pause:3.9
ExecReload=/bin/kill -s HUP $MAINPID
TimeoutSec=0
RestartSec=2
Restart=always
StartLimitBurst=3
StartLimitInterval=60s
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
Delegate=yes
KillMode=process
[Install]
WantedBy=multi-user.target
EOF

# cri-docker.socket
cat <<EOF | sudo tee /usr/lib/systemd/system/cri-docker.socket
[Unit]
Description=CRI Docker Socket for the API
PartOf=cri-docker.service

[Socket]
ListenStream=%t/cri-dockerd.sock
SocketMode=0660
SocketUser=root
SocketGroup=docker

[Install]
WantedBy=sockets.target
EOF

# 设置开机启动
systemctl daemon-reload
systemctl enable cri-docker.service
systemctl restart cri-docker.service
# 检查状态
systemctl is-active cri-docker
```



#### 安装harbor

```
# 安装docker-compose
yum install -y epel-release
yum install -y docker-compose

# 下载harbor
wget https://github.com/goharbor/harbor/releases/download/v2.5.3/harbor-offline-installer-v2.5.3.tgz
# 国内镜像
# wget https://ghproxy.com/https://github.com/goharbor/harbor/releases/download/v2.5.3/harbor-offline-installer-v2.5.3.tgz
tar -zxf harbor-offline-installer-v2.5.3.tgz -C /data/
cd /data/harbor

# 装载镜像
docker load < harbor.v2.5.3.tar.gz
cp harbor.yml.tmpl harbor.yml
# 根据实际情况修改
# 1. 仓库主机名
# 2. 禁用https
# 3. 指定数据目录

# 生成配置
./prepare

# 安装
./install.sh

docker-compose ps


cat <<EOF | sudo tee /etc/systemd/system/harbor.service
[Unit]
Description=Harbor
After=docker.service systemd-networkd.service systemd-resolved.service
Requires=docker.service
Documentation=http://github.com/vmware/harbor

[Service]
Type=simple
Restart=on-failure
RestartSec=10
ExecStart=/usr/bin/docker-compose --file /data/harbor/docker-compose.yml up
ExecStop=/usr/bin/docker-compose --file /data/harbor/docker-compose.yml down

[Install]
WantedBy=multi-user.target
EOF

# 加载配置
systemctl daemon-reload
# 设置开机启动
systemctl enable harbor
# 启动服务
systemctl start harbor
# 检查状态
systemctl status harbor

```



登录 harbor http://registry.k8s.com ，创建用户和项目。



上传镜像

```
# 打标签
docker tag nginx:latest registry.k8s.com/library/nginx:1.21.5

# 登录
docker login registry.k8s.com      
Username: guest
Password: Aa123456

# 上传镜像
docker push registry.k8s.com/library/nginx:1.21.5

# 拉镜像
docker pull registry.k8s.com/library/nginx:1.21.5
```



#### 初始化k8s集群

```
# 配置源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装
yum install -y kubelet-1.25.3 kubeadm-1.25.3 kubectl-1.25.3

# 下载镜像
images=$(kubeadm config images list --kubernetes-version=1.25.3 | awk -F "/" '{print $NF}')
for i in ${images} 
do   
  docker pull registry.aliyuncs.com/google_containers/$i
  docker tag registry.aliyuncs.com/google_containers/$i registry.k8s.com/google_containers/$i
  docker push registry.k8s.com/google_containers/$i
  docker rmi registry.aliyuncs.com/google_containers/$i
done
```



master节点

```
kubeadm init \
--kubernetes-version=1.25.3 \
--apiserver-advertise-address=192.168.93.201 \
--image-repository=registry.k8s.com/google_containers \
--pod-network-cidr="10.244.0.0/16" \
--service-cidr="10.96.0.0/12" \
--ignore-preflight-errors=Swap \
--cri-socket=unix:///var/run/cri-dockerd.sock
```

node节点

初始化完成，按照 master 命令行复制。

```
kubeadm join 192.168.67.201:6443 \
--token y8pzmr.3gde2xw059szwfcm \
--discovery-token-ca-cert-hash sha256:8f953fc473f666dd0067f2df50567a2e069658d1bfd354910e8b0220a44ad7f3 \
--cri-socket=unix:///var/run/cri-dockerd.sock
```

认证

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

验证

```
kubectl get nodes
```

配置网络

参考 https://kubernetes.io/docs/concepts/cluster-administration/addons/

```
mkdir -p /data/kubernetes/network/flannel
cd /data/kubernetes/network/flannel
wget https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml


docker pull flannel/flannel:v0.22.2 
docker pull flannel/flannel-cni-plugin:v1.2.0
docker tag flannel/flannel:v0.22.2 registry.k8s.com/flannel/flannel:v0.22.2
docker tag flannel/flannel-cni-plugin:v1.2.0 registry.k8s.com/flannel/flannel-cni-plugin:v1.2.0
docker push registry.k8s.com/flannel/flannel:v0.22.2
docker push registry.k8s.com/flannel/flannel-cni-plugin:v1.2.0
```

修改 kube-flannel.yml ，替换官方仓库为 harbor 地址。

```
sudo sed -i 's/docker.io/registry.k8s.com/g' kube-flannel.yml
```

创建 flanel

```
# 创建
kubectl apply -f kube-flannel.yml

# 查看namespace下的pod
kubectl get pod -n kube-flannel

# 查看集群node
kubectl get node
```



最后再确认一下 kubelet，cri-docker，docker 3个服务要设置成开机启动，否则下次重启机器后 k8s 集群无法正常启动。

```
systemctl enable kubelet cri-docker docker
```

至此，k8s 集群环境搭建完成。



如果初始化失败或集群有问题，可以通过 reset 命令重置，master 和 node 节点都要执行。

```
kubeadm reset --cri-socket=unix:///var/run/cri-dockerd.sock
```



#### 后置操作

##### 自动补全

```
echo "source <(kubectl completion bash)" >> ~/.bashrc 
echo "source <(kubeadm completion bash)" >> ~/.bashrc 
source ~/.bashrc
```



##### 非master节点配置kubectl

```
# 从master拷贝配置文件到node节点
scp /etc/kubernetes/admin.conf k8s-node1:/etc/kubernetes/
# 配置环境变量
echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> ~/.bashrc 
source ~/.bashrc
```



### 常用命令

#### Pod

创建pod

```
# kubectl run my-nginx --image=registry.k8s.com/library/nginx:1.21.5
pod/my-nginx created
```

查看pod

```
# kubectl get pod
NAME       READY   STATUS    RESTARTS   AGE
my-nginx   1/1     Running   0          56s
```

查看pod创建过程

```
# kubectl describe pod nginx
Name:             nginx
Namespace:        default
Priority:         0
Service Account:  default
Node:             k8s-node2/192.168.93.203
Start Time:       Mon, 18 Sep 2023 23:17:22 +0800
Labels:           run=nginx
Annotations:      <none>
Status:           Running
IP:               10.244.2.9
IPs:
  IP:  10.244.2.9
Containers:
  nginx:
    Container ID:   docker://18b65f77faf6dc944fb84b8e8a6d5b1b0b992598814ff1449bd48a4a7d4f84ba
    Image:          registry.k8s.com/library/nginx:1.21.5
    Image ID:       docker-pullable://registry.k8s.com/library/nginx@sha256:ee89b00528ff4f02f2405e4ee221743ebc3f8e8dd0bfd5c4c20a2fa2aaa7ede3
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Mon, 18 Sep 2023 23:17:24 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8tcpm (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-8tcpm:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m38s  default-scheduler  Successfully assigned default/nginx to k8s-node2
  Normal  Pulled     3m38s  kubelet            Container image "registry.k8s.com/library/nginx:1.21.5" already present on machine
  Normal  Created    3m37s  kubelet            Created container nginx
  Normal  Started    3m37s  kubelet            Started container nginx
```

查看日志

```
# kubectl logs nginx
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: info: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: info: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Launching /docker-entrypoint.d/30-tune-worker-processes.sh
/docker-entrypoint.sh: Configuration complete; ready for start up
2023/09/18 15:17:24 [notice] 1#1: using the "epoll" event method
2023/09/18 15:17:24 [notice] 1#1: nginx/1.21.5
2023/09/18 15:17:24 [notice] 1#1: built by gcc 10.2.1 20210110 (Debian 10.2.1-6) 
2023/09/18 15:17:24 [notice] 1#1: OS: Linux 3.10.0-1160.71.1.el7.x86_64
2023/09/18 15:17:24 [notice] 1#1: getrlimit(RLIMIT_NOFILE): 1048576:1048576
2023/09/18 15:17:24 [notice] 1#1: start worker processes
2023/09/18 15:17:24 [notice] 1#1: start worker process 31
2023/09/18 15:17:24 [notice] 1#1: start worker process 32
10.244.2.1 - - [18/Sep/2023:15:18:30 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/7.29.0" "-"
```

进入容器

```
# kubectl exec -it nginx -- /bin/bash
root@nginx:/# curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
root@nginx:/# exit
exit
```

查看扩展信息

```
# kubectl get pod -o wide
NAME      READY   STATUS    RESTARTS   AGE     IP            NODE        NOMINATED NODE   READINESS GATES
nginx     1/1     Running   0          5m34s   10.244.2.9    k8s-node2   <none>           <none>
# curl 10.244.2.9
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

通过 yaml 创建容器

```
# kubectl get pod my-nginx -o yaml > 010_pod_nginx.yaml
# kubectl apply -f 010_pod_nginx.yaml
pod/nginx created
# kubectl delete -f nginx.yaml 
pod "nginx" deleted
```

#### Deployment

创建deployment

```
# kubectl create deployment nginx --image=registry.k8s.com/library/nginx:1.21.5
deployment.apps/nginx created
# kubectl get deployment
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           61s
# kubectl get rs
NAME               DESIRED   CURRENT   READY   AGE
nginx-85bb584974   1         1         1       2m5s
# kubectl delete deployments.apps nginx 
deployment.apps "nginx" deleted
```

使用 yaml 创建 deployment

```
# kubectl create deployment nginx --image=registry.k8s.com/library/nginx:1.21.5 --dry-run=client -o yaml > 020_deploy_nginx.yaml
# kubectl create -f 020_deploy_nginx.yaml 
deployment.apps/nginx created
```

修改 deployment 配置

```
# kubectl edit deployments.apps nginx 
deployment.apps/nginx edited
```

动态调整 pod 数量

```
# kubectl scale deployment nginx --replicas=5
deployment.apps/nginx scaled
```

#### Namespace

创建 namespace

```
# kubectl create namespace my-ns
namespace/my-ns created
# kubectl get ns
NAME              STATUS   AGE
default           Active   14h
kube-flannel      Active   5h6m
kube-node-lease   Active   14h
kube-public       Active   14h
kube-system       Active   14h
my-ns             Active   35s
# kubectl delete namespaces my-ns
namespace "my-ns" deleted
```

通过 yaml 创建 namespace 和 pod

```
# kubectl apply -f my-ns.yaml  
namespace/my-ns created
pod/my-nginx-by-yaml created
# kubectl get pod -n my-ns
NAME               READY   STATUS    RESTARTS   AGE
my-nginx-by-yaml   1/1     Running   0          30s
```



```
# kubectl expose deployment  nginx --port=80
service/nginx exposed
# kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   5d16h
nginx        ClusterIP   10.108.211.75   <none>        80/TCP    11s
```



#### Service

```
# kubectl expose deploy nginx --port=80
service/nginx exposed

# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   93m
nginx        ClusterIP   10.99.1.55   <none>        80/TCP    13s

# kubectl describe svc nginx 
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.99.1.55
IPs:               10.99.1.55
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.16:80
Session Affinity:  None
Events:            <none>

# kubectl get pod -o wide
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE        NOMINATED NODE   READINESS GATES
nginx                   1/1     Running   0          42m   10.244.1.13   k8s-node1   <none>           <none>
nginx-5954f9f48-rl4g2   1/1     Running   0          15m   10.244.1.16   k8s-node1   <none>           <none>
```



```
# kubectl apply -f 030_svc_nginx.yaml 
service/nginx created
[root@k8s-master kubernetes]# kubectl get svc -o wide
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE     SELECTOR
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   5d17h   <none>
nginx        ClusterIP   10.105.130.28   <none>        80/TCP    23s     app=nginx
```


### 服务发现

#### Service

在 k8s 中，Service 可以将一组具有相同功能的 pod 提供一个统一的入口地址，并将请求进行负载分发到各个 pod 上。

Service 有四种类型，包括：

1. ClusterIp：这是默认类型，自动分配一个仅 Cluster 内部可以访问的虚拟 IP 。
2. NodePort：在 ClusterIP 基础上为 Service 在每台机器上绑定一个端口，这样就可以通过 NodePort 来访问该服务。
3. LoadBalancer：在 NodePort 的基础上，借助 Cloud Provider 创建一个外部负载均衡器，并将请求转发到NodePort 。
4. ExternalName：把集群外部的服务引入到集群内部来，在集群内部直接使用。 没有任何类型代理被创建，这只有 Kubernetes 1.7 或更高版本的 kube-dns 才支持。



#### Ingress



#### ConfigMap



#### Secret

在 Kubernetes 中，ConfigMap 通常用于存储配置信息，如果配置信息中有用户名和密码等敏感信息，直接在在 ConfigMap 中并不安全。为了安全起见，你可以使用 Kubernetes 的 Secret 来存储敏感信息，然后在 ConfigMap 中通过环境变量引用这些 Secret。

```
apiVersion: v1  
kind: Secret  
metadata:  
  namespace: kube-system  
  name: kibana-elasticsearch-credentials  
type: Opaque  
data:  
  elasticsearch-username: YVSVRFVDU0VSVENBUVVTRU5BTUU= # base64 编码的用户名  
  elasticsearch-password: RVNBUFRJTkdIRUNIVEVSREVSQ0VSU0U= # base64 编码的密码
```

使用 `echo -n "your_username" | base64` 和 `echo -n "your_password" | base64` 来生成这些值。

创建





### Helm