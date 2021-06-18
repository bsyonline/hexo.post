---
title: install gitlab using docker
tags:
  - Docker
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2019-04-28 00:08:33
thumbnail:
---



#### 下载镜像

```
docker pull gitlab/gitlab-ce
```

#### 启动

```
docker run -d --hostname gitlab --publish 443:443 --publish 80:80 --publish 22:22 --name gitlab --restart always --volume D:\data\gitlab\config:/etc/gitlab --volume D:\data\gitlab\logs:/var/log/gitlab --volume D:\data\gitlab\data:/var/opt/gitlab gitlab/gitlab-ce
```

