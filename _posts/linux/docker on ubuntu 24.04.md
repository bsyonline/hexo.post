---
title: docker on ubuntu 24.04
date: 2016-10-24 18:31:44
tags:
  - Ubuntu
category:
  - Linux
thumbnail: 
author: bsyonline
lede: 没有摘要
---




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

```sh
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "registry-mirrors": ["https://xxx.mirror.aliyuncs.com"],
  "dns": ["114.114.114.114", "8.8.8.8"],
  "insecure-registries": ["registry.k8s.com"]
}
EOF

# 重启 docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```
