---
title: N8N
tags:
  - AI
category:
  - AI
author: bsyonline
lede: 没有摘要
date: 2018-05-28 15:32:21
thumbnail:
---


```
docker run -it --rm \
--name n8n \
-p 5678:5678 \
--add-host=localhost:host-gateway \
-e GENERIC_TIMEZONE="Asia/Shanghai" \
-e TZ="Asia/Shanghai" \
-e N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
-e N8N_RUNNERS_ENABLED=true \
-e N8N_SECURE_COOKIE=false \
-v n8n_data:/home/node/.n8n \
docker.n8n.io/n8nio/n8n:1.108.1
```






