---
title: 单机搭建 elk + logback 环境
date: 2016-08-16 15:53:17
tags:
 - Elastic
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---
elk是开源日志分析平台，由 [elastic](http://www.elastic.co) 公司的三款开源产品组成。


### 1. 准备工作
下载elasticsearch、logstash、kibana。

### 2. 安装

#### 2.1 elasticsearch

解压
```
tar -zxf elasticsearch-2.3.1.tar.gz
```
***elasticsearch-2.3.1/config/elasticsearch.yml***
```
cluster.name: es
node.name: node1
network.host: node1
discovery.zen.ping.unicast.hosts: ["node1"]
```
启动
```
bin/elasticsearch
```
#### 2.2 logstash
解压
```
logstash-2.3.2.tar.gz
```
新建配置文件
```
mkdir etc
cd etc
```
***my.conf***
```
input {

    stdin {
    }

    redis {
        batch_count => 1
        data_type => "list"
        key => "logstash"
        host => "127.0.0.1"
        port => 6379
        db => 0
        threads => 1
    }

#    tcp {
#        host  => "127.0.0.1"
#        port  => 5678
#        codec => "line"
#    }

    log4j {
        mode => "server"
        host  => "127.0.0.1"
        port => 56789
        type => "log4j"
    }
}

output {
    stdout {
        codec => rubydebug
    }

    elasticsearch {
        hosts => ["node1:9200"]
        index => "logstash"
        document_type => "log"
        workers => 1
        flush_size => 20000
        idle_flush_time => 10
        template_overwrite => true
    }
}

```
启动
```
bin/logstash agent -f ../etc/my.conf
```
#### 2.3 kibana
解压
```
tar -zxf kibana-4.5.1-linux-x64.tar.gz
```
***config/kibana.yml***
```
elasticsearch.url: "http://node1:9200"
```
启动
```
bin/kibana
```
### 3.收集日志
#### 3.1 使用 log4j 收集日志
logstash 有 log4j 的 input 插件，所以使用 log4j 可以很容易将日志收集到 logstash 。

***Log4jTest.java***
```java
import org.apache.log4j.Logger;
public class Log4jTest {
    private static Logger logger = Logger.getLogger(Log4jTest.class);

    public static void main(String[] args) {

        logger.debug("hello logstash, this is a message from log4j");

    }
}
```
***log4j.properties***
```
log4j.appender.stdout=org.apache.log4j.ConsoleAppender
log4j.appender.stdout.layout=org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern=%d %p %t %c : %m%n

log4j.appender.file=org.apache.log4j.RollingFileAppender
log4j.appender.file.file=../log/test.log
log4j.appender.file.maxFileSize=1024
log4j.appender.file.layout=org.apache.log4j.PatternLayout
log4j.appender.file.layout.ConversionPattern=%d %p %t %c : %m%n

# logstash配置
log4j.appender.logstash=org.apache.log4j.net.SocketAppender
log4j.appender.logstash.port=56789
log4j.appender.logstash.remoteHost=127.0.0.1

log4j.rootLogger=debug,stdout,file,logstash
```
运行程序可在 kibana 中看到日志。

#### 3.2 使用 logback 收集日志
logstash 没有 logback 的插件，可以使用 tcp 方式收集日志（效果不好，官方也不建议在生产环境使用tcp方式）。
##### 3.2.1 tcp 方式
***logback.xml***
```
<configuration>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoder 默认配置为PatternLayoutEncoder -->
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="SOCKET" class="ch.qos.logback.classic.net.SocketAppender">
        <remoteHost>127.0.0.1</remoteHost>
        <port>5678</port>
        <reconnectionDelay>10000</reconnectionDelay>
        <includeCallerData>true</includeCallerData>
    </appender>

    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="SOCKET" />
    </root>

</configuration>
```

##### 3.2.2 logback + redis

使用 redis 作为消息队列，需要用到 logback-rides 的开源包。

安装redis

```
tar -zxf redis-3.2.0.tar.gz
cd redis-3.2.0/
sudo make
sudo make install
```
注：如果logstash和redis不在同一台机器，需要修改***redis.conf***
```
#bind 127.0.0.1
protected-mode no
```
启动
```
/usr/local/bin/redis-server
```

***pom.xml***
```
<dependency>
    <groupId>com.cwbase</groupId>
    <artifactId>logback-redis-appender</artifactId>
    <version>1.1.5</version>
</dependency>
```

***LogbackTest.java***
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class LogbackTest {

    static Logger logger = LoggerFactory.getLogger(LogbackTest.class);
    public static void main(String[] args) {
        logger.info("this log come from slf4j.");
    }
}
```
***logback.xml***
```
<configuration>

    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <!-- encoder 默认配置为PatternLayoutEncoder -->
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="LOGSTASH" class="com.cwbase.logback.RedisAppender">
        <source>mySource</source>
        <sourcePath>mySourcePath</sourcePath>
        <type>myApplication</type>
        <tags>production</tags>
        <host>127.0.0.1</host>
        <port>6379</port>
        <key>logstash</key>
    </appender>

    <root level="INFO">
        <appender-ref ref="STDOUT" />
        <appender-ref ref="LOGSTASH" />
    </root>

</configuration>
```
