---
title: Building Log Collection System
tags:
  - ELK
category:
  - Microservices
author: bsyonline
lede: 没有摘要
date: 2020-05-02 17:06:57
thumbnail:
---

日志收集方式有很多，本文主要介绍利用 Kafka 和 ELK 来进行日志收集。

<img src="https://s1.ax1x.com/2020/05/02/JvOEse.png" alt="20200502190811" border="0"  style="width:600px">
设计思路很简单：
1. 首先通过日志框架，将日志分级并存储到文件。
2. 通过 Filebeat 将文件内容放到 Kafka 中。
3. Logstash 对 Kafka 中的日志进行消费，过滤出需要的日志，写到 Elasticsearch 中。
4. 最后用 Kibana 进行展示。


使用的环境是 Centos7 + logback + Kafka_2.11-2.4.1 + ELK Stack 6.8.8 。这里简单介绍一下 ELK Stack 。ELK Stack 就是 ELK + Beats 。ELK 是三个开源项目的首字母缩写，这三个项目分别是：Elasticsearch 、Logstash 和 Kibana 。Elasticsearch 是一个搜索和分析引擎。Logstash  是服务器端数据处理管道，能够同时从多个来源采集数据，转换数据，然后将数据发送到诸如 Elasticsearch 等“存储库”中。Kibana 则可以让用户在 Elasticsearch  中使用图形和图表对数据进行可视化。Beats 是轻量型的单一功能数据采集器，Filebeat 是 Beats 中的一个用于文件的采集器。

#### 日志
第一步首先要选择一个日志框架，日志框架有很多，根据自己习惯选就好了。这里用的是 Logback ，它是 SLF4J 的直接实现，SpringBoot 默认使用的也是它。接下来就是要将日志分级，把全量 Log 和 Error Log 分别存储，以 logback-spring.xml 为例。
```
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <contextName>logback</contextName>
    <property name="app.name" value="log-collection"/>
    <property name="log.path" value="./logs/${app.name}"/>
    <!--输出到控制台-->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <layout>
            <Pattern>[%X{host}] [%X{ip}] [%X{appName}] %d{yyyy-MM-dd'T'HH:mm:ss.SSS} %level ${PID:- } %thread %logger - %msg ##'%ex'%n</Pattern>
        </layout>
    </appender>

    <!--输出到文件-->
    <appender name="infoFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/app.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 按天轮转 -->
            <fileNamePattern>${log.path}/app-%d{yyyy-MM-dd}-%i.zip</fileNamePattern>
            <!-- 单个文件最大 500M -->
            <maxFileSize>500MB</maxFileSize>
            <!-- 保存 30 天的历史记录，最大大小为 30GB -->
            <maxHistory>30</maxHistory>
            <totalSizeCap>3GB</totalSizeCap>
        </rollingPolicy>
        <layout>
            <Pattern>[%X{host}] [%X{ip}] [%X{appName}] %d{yyyy-MM-dd'T'HH:mm:ss.SSS} %level ${PID:- } %thread %logger - %msg ##'%ex'%n</Pattern>
        </layout>
    </appender>
    <appender name="errorFile" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.path}/error.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 按天轮转 -->
            <fileNamePattern>${log.path}/error-%d{yyyy-MM-dd}-%i.zip</fileNamePattern>
            <!-- 单个文件最大 500M -->
            <maxFileSize>500MB</maxFileSize>
            <!-- 保存 30 天的历史记录，最大大小为 30GB -->
            <maxHistory>30</maxHistory>
            <totalSizeCap>3GB</totalSizeCap>
        </rollingPolicy>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <layout>
            <Pattern>[%X{host}] [%X{ip}] [%X{appName}] %d{yyyy-MM-dd'T'HH:mm:ss.SSS} %level ${PID:- } %thread %logger - %msg ##'%ex'%n</Pattern>
        </layout>
    </appender>

    <root level="info">
        <appender-ref ref="console"/>
        <appender-ref ref="infoFile"/>
        <appender-ref ref="errorFile"/>
    </root>

</configuration>
``` 
这里需要特殊说明的是 Layout.Pattern 。一行日志通过空格分隔，前 3 个是自定义 MDC 属性，第 4 部分是 UTC Timestamp ，第 5 部分是日志级别，第 6 部分是进程号，第 7 部分是线程名字，第 8 部分是类的名字，第 9 部分是日志信息，最后一部分是异常信息。由于异常堆栈是多行，所以通过 ”‘“ 包起来便于后续处理。
Logback 更多内容参考 [http://www.logback.cn/](http://www.logback.cn/) 。

#### Filebeat
Filebeat 用来对文件内容进行收集，将其和应用程序部署在同一台机器。下载解压之后修改配置文件 filebeat.yml 。
```
filebeat.inputs:

- type: log
  enabled: true
  paths:
    - /logs/log-collection/app.log 	# 指定日志文件
  document_type: "app-log"
  multiline:
    pattern: '^\['					# 以 ”[“ 开头，不是 [ 开头则合并到上一行，最大2000行
    negate: true
    match: after
    max_lines: 2000
    timeout: 2s
  fields:
    logbiz: log-collection			# 应用名称
    log_topic: app-log-collection	# Kafka 的 Topic
    evn: dev						# 环境
    
- type: log
  enabled: true
  paths:
    - /logs/log-collection/error.log
  document_type: "error-log"
  multiline:
    pattern: '^\['
    negate: true
    match: after
    max_lines: 2000
    timeout: 2s
  fields:
    logbiz: log-collection
    log_topic: error-log-collection
    evn: dev

output.kafka:						# 输出到 Kafka
  enabled: true
  hosts: ["192.168.0.106:9092"]		# Kafka Broker 地址
  topic: '%{[fields.log_topic]}'	# Kafka Topic ，动态获取上边配置的 Topic
  partition.hash:
    reachable_only: true
  compression: gzip
  max_message_bytes: 1000000
  required_acks: 1
logging.to_files: true
``` 
启动，带 -e 可以看到日志。
```
./filebeat -e
```

#### Kafka

Kafka 安装配置参考 [Kafka Quickstart](../../../../2020/05/02/kafka-quickstart/) 。

创建 Filebeat 要用到的 Topic 。
```
bin/kafka-topics.sh --zookeeper 192.168.0.206:2181 --create --topic app-log-collection --partitions 1 --replication-factor 1
bin/kafka-topics.sh --zookeeper 192.168.0.206:2181 --create --topic error-log-collection --partitions 1 --replication-factor 1
```

启动 Filebeat 之后，如果配置正常，在 Kafka 的 logs 目录可以看到对应 topic 的日志。

#### Logstash

Lostash 作用是消费 Kafka 中的日志。下载解压之后修改配置文件 config/logstash.conf 。
```
# 输入，可以有多个
input {
  # Kafka 的配置
  kafka {
	topics_pattern => "app-log-.*" 				# Kafka 的 Topic 模式
	bootstrap_servers => "192.168.0.206:9092"	# Kafka broker 地址
	codec => json
	consumer_threads => 4
	decorate_events => true
	group_id => "app-log-group"
  }
  kafka {
	topics_pattern => "error-log-.*"
	bootstrap_servers => "192.168.0.206:9092"
	codec => json
	consumer_threads => 4
	decorate_events => true
	group_id => "error-log-group"
  }
}

filter {
  ruby {
    # 将本地时间赋给变量 index_time ，在 ES 中按天创建索引会用到
	# Logstash 使用的 UTC 时间，所以会比东八区晚 8 小时，这里进行时区转换
	code => "event.set('index_time', event.timestamp.time.localtime.strftime('%Y.%m.%d'))"
  }
  
  if "app-log" in [fields][log_topic] { 	# Filebeat 中的变量
	grok {
	  # 针对日志格式进行解析，变量名字无所谓，位置需要对应
	  match => ["message", "\[%{DATA:host}\] \[%{DATA:ip}\] \[%{DATA:appName}\] %{NOTSPACE:currentDateTime} %{NOTSPACE:level} %{NOTSPACE:pid} %{NOTSPACE:thread} %{NOTSPACE:class} - %{DATA:msgInfo} ##(\'\'|%{QUOTEDSTRING:throwable})"]
	}
	# 还是时区的问题，ELK 缺省使用的是 UTC ，所以在这把时区加上，在 ES 和 Kibana 里就都按指定时区了
	mutate {
	  gsub => ["currentDateTime", "[+]", "T"]
	} 
	mutate{
	  replace => ["currentDateTime","%{currentDateTime}+08:00"]
	}
  }
  
  if "error-log" in [fields][log_topic] {
	grok {
	  match => ["message", "\[%{DATA:host}\] \[%{DATA:ip}\] \[%{DATA:appName}\] %{NOTSPACE:currentDateTime} %{NOTSPACE:level} %{NOTSPACE:pid} %{NOTSPACE:thread} %{NOTSPACE:class} - %{DATA:msgInfo} ##(\'\'|%{QUOTEDSTRING:throwable})"]
	}
	mutate {
	  gsub => ["currentDateTime", "[+]", "T"]
	} 
	mutate{
	  replace => ["currentDateTime","%{currentDateTime}+08:00"]
	}
  }
}

# 输出
output {
  # 输出到控制台
  stdout { 
	codec => rubydebug
  }
}

output {
  # 输出到 ES
  if "app-log" in [fields][log_topic] {
    elasticsearch {
	  hosts => ["192.168.0.201:9200"]						# ES 地址
	  index => "app-log-%{[fields][logbiz]}-%{index_time}"	# ES 索引的名字，按天生成索引
	  sniffing => true										# 嗅探模式
	  template_overwrite => true	  							
	}
  }
  
  if "error-log" in [fields][log_topic] {
    elasticsearch {
	  hosts => ["192.168.0.201:9200"]
	  index => "error-log-%{[fields][logbiz]}-%{index_time}"
	  sniffing => true
	  template_overwrite => true	  
	}
  }
}
```
配置完成后启动
```
./bin/logstash -f ./config/logstash.conf
```
更多 Logstash 内容参考 [Logstash 最佳实践](http://doc.yonyoucloud.com/doc/logstash-best-practice-cn/index.html) 。
#### Elasticsearch

Elasticsearch 集群搭建参考 [Elasticsearch Quickstart](../../../../2020/05/02/elasticsearch-quickstart/) 。

#### Kibana
下载解压之后在 config/kibana.yml 中配置 ES 的地址就可以运行了。
```
./bin/kibana
```
启动之后访问 [http://locahost:5601](http://locahost:5601) 。
先去 Management > Kibana > Advanced Settings 里修改时区为东八区，然后创建 Index Patterns 。
<img src="https://s1.ax1x.com/2020/05/02/JvOVqH.png" alt="20200502183933" border="0">
在 Discover 中就可以看到日志了。

#### 错误告警
通过 Elasticsearch Watcher 可以进行错误告警。Watcher 就是一个定时任务，通过 Watcher API 可以创建 Watcher ，比如创建一个 Watcher 每 10s 扫描一次 error-log 索引。
```
PUT _xpack/watcher/watch/applog_error_watcher
{
  # 触发器
  "trigger":{
    "schedule":{
      "interval":"10s"
    }
  },
  "input":{
    "search":{
      "request":{
        "indices":[
          "error-log-*"
        ],
        "body":{
          "size":0,
          "query":{
            "bool":{
              "must":{
                "term":{
                  "level.keyword":"ERROR"
                }
              },
              "filter":{
                "range":{
                  "currentDateTime":{
                    "gt":"now-30s",
                    "lt":"now"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  # 条件：查询命中条数大于0触发
  "condition":{
      "compare":{
          "ctx.payload.hits.total":{
              "gt":0
          }
      }
  },
  # 查询数据
  "transform":{
    "search":{
      "request":{
        "indices":[
          "error-log-*"
        ],
        "body":{
          "size":10,
          "query":{
            "bool":{
              "must":{
                "term":{
                  "level.keyword":"ERROR"
                }
              },
              "filter":{
                "range":{
                  "currentDateTime":{
                    "gt":"now-30s",
                    "lt":"now"
                  }
                }
              }
            }
          }
        }
      }
    }
  },
  # 处理动作，这里是注册了一个webhook，然后进行回调
  "actions":{
    "test_error":{
      "throttle_period":"1m",
      "webhook":{
        "method":"POST",
        "url":"http://192.168.0.106:8080/watch",
        "body":"watcher test"
      }
    }
  }
}
```


