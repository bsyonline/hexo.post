---
title: harbor installation on Ubuntu 24.04
date: 2017-04-01 15:05:26
tags:
  - Ubuntu
category:
  - Linux
thumbnail: 
author: bsyonline
lede: 没有摘要
---



安装docker-compose

```sh
sudo add-apt-repository universe
sudo apt update
sudo apt-get install -y pipx
sudo pipx install -y setuptools
sudo pipx install -y 'urllib3<2'
sudo apt install docker-compose

docker-compose --version
```

下载harbor

```sh
wget https://github.com/goharbor/harbor/releases/download/v2.5.3/harbor-offline-installer-v2.5.3.tgz
# 国内镜像
# wget https://ghproxy.com/https://github.com/goharbor/harbor/releases/download/v2.5.3/harbor-offline-installer-v2.5.3.tgz
tar -zxf harbor-offline-installer-v2.5.3.tgz -C /data/
cd /data/harbor
```

装载镜像

```sh
docker load < harbor.v2.5.3.tar.gz
```

修改配置

```sh
cp harbor.yml.tmpl harbor.yml
# 根据实际情况修改
# 1. 仓库主机名
# 2. 禁用https
# 3. 指定数据目录

# 生成配置
./prepare
```


安装

```sh
./install.sh

docker compose ps
```

设置开机启动

```sh
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
ExecStart=docker compose --file /data/harbor/docker-compose.yml up
ExecStop=docker compose --file /data/harbor/docker-compose.yml down

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


登录 harbor http://reg.k8s.me ，创建用户和项目。

