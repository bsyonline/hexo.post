---
title: Rancher 搭建 ELK 日志收集系统小记
date: 2017-02-17 16:05:40
tags:
 - Elastic
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

ELK 是 elastic 公司提供的一套开源的实时日志分析系统。经过一段时间的学习，在 Rancher 上搭建了一套，在此将整个搭建过程做一个总结。

<!-- more -->

### 环境说明

使用环境：
```
Rancher         v1.3.1
docker          v1.13.1
elasticsearch   v2.3
logstash        v2.3
kibana          v4.5
redis           v3.0
```

### 结构

目前的需求是能够将多个应用的响应类的日志统计收集起来集中查看，所以我使用了一个简化的方式来搭建 ELK 系统。

应用程序通过 log 将日志写到 redis ，logstash 从 redis 收集日志，写到 elasticsearch ，kibana 从 elasticsearch 中获取数据展现。

![https://raw.githubusercontent.com/bsyonline/pic/master/20181014/elk.png](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/elk.png)

### 搭建步骤

#### 准备镜像

下载镜像
```shell
docker pull logstash:2.3
docker pull elasticsearch:2.3
docker pull kibana:4.5
docker pull redis:3.0
```
打标签
```shell
docker tag logstash:2.3 reg.dockcloud.cn/riskbell/logstash:2.3
docker tag kibana:4.5 reg.dockcloud.cn/riskbell/kibana:4.5
docker tag redis:3.0 reg.dockcloud.cn/riskbell/redis:3.0
docker tag elasticsearch:2.3 reg.dockcloud.cn/riskbell/elasticsearch:2.3
```
上传本地仓库
```shell
docker push reg.dockcloud.cn/riskbell/logstash:2.3
docker push reg.dockcloud.cn/riskbell/kibana:4.5
docker push reg.dockcloud.cn/riskbell/redis:3.0
docker push reg.dockcloud.cn/riskbell/elasticsearch:2.3
```

#### rancher 配置
##### redis

redis 需要向外暴露端口给应用程序。

![https://raw.githubusercontent.com/bsyonline/pic/master/20181014/2020170217164234.png](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/20170217164234.png)
##### elasticsearch

elasticsearch 也不需要使用别的服务，为了能从浏览器查看数据，也向外暴露端口。

![https://raw.githubusercontent.com/bsyonline/pic/master/20170217164654.png](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/20170217164654.png)

##### kibana

kibana 需要 link elasticsearch 的服务，也要向外暴露端口。

![https://raw.githubusercontent.com/bsyonline/pic/master/20170217164908.png](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/20170217164908.png)

##### logstash

logstash 的 input 是 redis ， output 是 elasticsearch ，需要在配置文件中指定。

```
input {

    stdin {
    }

    redis {
        batch_count => 1
        data_type => "list"
        key => "logstash"
        host => "redis"               #host
        port => 6379              
        db => 0
        threads => 1
    }
}

output {
    stdout {
        codec => rubydebug
    }

    elasticsearch {
        hosts => ["elasticsearch"]    #host:port
        index => "logstash-%{+YYYY.MM.dd}"
        document_type => "log"
        workers => 1
        flush_size => 20000
        idle_flush_time => 10
        template_overwrite => true
    }
}
```

应为使用 rancher ，所以这份配置文件的 host 位置没有写 ip ，而是 rancher 中的服务名字。由于 link 服务走的是 overlay 网络，所以，端口号也是内部的，配置文件在启动命令中指定。

![https://raw.githubusercontent.com/bsyonline/pic/master/20170217165818.png](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/20170217165818.png)

这样就完成了 ELK 系统的搭建。

![https://raw.githubusercontent.com/bsyonline/pic/master/Screenshot%20from%202017-02-17%2016-58-56.png](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/20170217165856.png)

### compose
docker-compose.yml

```
version: '2'
services:
  logstash:
    image: dev.dockcloud.cn/riskbell/logstash:2.3
    stdin_open: true
    volumes:
    - /opt/riskbell-elk/conf:/conf
    tty: true
    links:
    - elasticsearch:elasticsearch
    - redis:redis
    command:
    - logstash
    - -f
    - /conf/riskbell.conf
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: hostname=d21
  elasticsearch:
    image: dev.dockcloud.cn/riskbell/es:2.3
    stdin_open: true
    tty: true
    ports:
    - 19200:9200/tcp
    - 19300:9300/tcp
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: hostname=d21
  kibana:
    image: dev.dockcloud.cn/riskbell/kibana:4.5
    stdin_open: true
    tty: true
    links:
    - elasticsearch:elasticsearch
    ports:
    - 15601:5601/tcp
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: hostname=d21
  redis:
    image: dev.dockcloud.cn/riskbell/redis:latest
    stdin_open: true
    tty: true
    ports:
    - 16379:6379/tcp
    labels:
      io.rancher.container.pull_image: always
      io.rancher.scheduler.affinity:host_label: hostname=d21
```
rancher-compose.yml
```
version: '2'
services:
  logstash:
    scale: 1
    start_on_create: true
  elasticsearch:
    scale: 1
    start_on_create: true
  kibana:
    scale: 1
    start_on_create: true
  redis:
    scale: 1
    start_on_create: true
```
