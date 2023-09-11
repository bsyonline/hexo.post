---
title: Build dolphinscheduler Dev Environment
tags:
  - bigdata
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2018-05-31 23:20:09
thumbnail:
---





### 安装zk



### 下载源码



### 修改数据源

修改工程根目录 pom.xml 中的 mysql 依赖 scope 。

```
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <version>${mysql.connector.version}</version>
    <scope>compile</scope>
</dependency>
```

修改 dolphinscheduler-dao 数据库链接信息。

使用 dolphinscheduler/dolphinscheduler-dao/src/main/resources/sql/dolphinscheduler_mysql.sql 初始化数据库。

### 修改日志配置

修改 dolphinscheduler/dolphinscheduler-server/src/main/resources/logback-master.xml，dolphinscheduler/dolphinscheduler-server/src/main/resources/logback-worker.xml， dolphinScheduler/dolphinscheduler-api/src/main/resources/logback-api.xml增加控制台输出。

```
<root level="INFO">
+   <appender-ref ref="STDOUT"/>
    <appender-ref ref="TASKLOGFILE"/>
    <appender-ref ref="WORKERLOGFILE"/>
</root>
```

### 启动服务

修改 dolphinscheduler/dolphinscheduler-server/src/main/resources/master.properties，dolphinscheduler/dolphinscheduler-server/src/main/resources/worker.properties，dolphinScheduler/dolphinscheduler-api/src/main/resources/application-api.properties ，增加 mysql 配置。

```
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/dolphinscheduler?useUnicode=true&characterEncoding=UTF-8&useSSL=false&serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=123456
```

运行 org.apache.dolphinscheduler.server.master.MasterServer，配置 *VM Options* `-Dlogging.config=classpath:logback-spring.xml -Ddruid.mysql.usePingMethod=false -Dspring.profiles.active=mysql`。

运行 org.apache.dolphinscheduler.server.master.WorkerServer，配置 *VM Options* `-Dlogging.config=classpath:logback-spring.xml -Ddruid.mysql.usePingMethod=false -Dspring.profiles.active=mysql`。

运行 org.apache.dolphinscheduler.api.ApiApplicationServer，配置 *VM Options* `-Dlogging.config=classpath:logback-spring.xml -Dspring.profiles.active=api,mysql`。

### 启动UI

```
cd dolphinscheduler-ui
npm install
npm run start
```

启动完成后，通过 http://localhost:8888 访问。默认账密 admin/dolphinscheduler123 。