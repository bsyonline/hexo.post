---
title: Introducing Nacos
tags:
  - Spring Cloud
category:
  - Spring Cloud
  - Microservices
author: bsyonline
lede: 没有摘要
date: 2020-04-19 17:09:54
thumbnail:
---

Nacos 是一个用于构建微服务的基础设施，提供服务发现、服务配置、元数据和流量管理功能。

#### 安装
目前推荐版本是 1.1.4 ，可以去 github 下载。直接下载二进制包速度太慢，可以使用源代码编译。
```
git clone https://github.com/alibaba/nacos.git
cd nacos/
mvn -Prelease-nacos -Dmaven.test.skip=true clean install -U  
```
编译完成，在 ```distribution/target/``` 中有编译好的二进制包，解压之后就可以使用命令启动。启动有单机和集群两种方式。
```
sh startup.sh -m standalone
```
不带参数为集群方式。

单机模式不需要什么配置，直接启动就可以。集群方式需要先修改 ```conf/cluster.conf``` 。
```
192.168.100.101:8848
192.168.100.102:8848
192.168.100.103:8848
```
集群至少需要 3 个节点。

启动之后访问 [http://localhost:8848/nacos/index.html](http://localhost:8848/nacos/index.html) 可以看到界面，默认用户名密码为：nacos/nacos 。

#### 添加获取配置
Nacos 通过 Namespace 、Group 、DataId 三元组来确定配置，Namespace 对应环境如 dev 、test 、prod ，Group 可以对应项目或工程，DataId 对应配置文件。Nacos 有多种方式，界面、OpenAPI 、SDK 。界面点点就好了，下面来说说另外两种方式。
##### OpenAPI 方式
发布
```
curl -XPOST "http://localhost:8848/nacos/v1/cs/configs" \
-H "Content-Type: application/x-www-form-urlencoded; charset=utf-8" \
--data-urlencode "dataId=application.properties" \
--data-urlencode "group=spring-cloud-alibaba-nacos" \
--data-urlencode "tenant=e400be7d-039f-460a-bbb9-d2a2752fcb9e" \
--data-urlencode "content=application.name=nacos"
```
>这里有一点需要注意，tenant 为 Namespace ，不能写名字，需要写 ID 。

获取
```
curl -XGET "http://localhost:8848/nacos/v1/cs/configs?dataId=application.properties&group=spring-cloud-alibaba-nacos&tenant=e400be7d-039f-460a-bbb9-d2a2752fcb9e"
```
监听
```
curl -XPOST "http://localhost:8848/nacos/v1/cs/configs/listener" -H "Long-Pulling-Timeout: 30000" -d "Listening-Configs=application.properties^2spring-cloud-alibaba-nacos^6db060a43bdf54674e916e7b3acca7fa^2e400be7d-039f-460a-bbb9-d2a2752fcb9e^1"
```

##### SDK
```
public class NacosSDK {
    public static void main(String[] args) throws NacosException, InterruptedException {
        String serverAddr = "localhost:8848";
        String dataId = "application.properties";
        String group = "spring-cloud-alibaba-nacos";
        String tenant = "e400be7d-039f-460a-bbb9-d2a2752fcb9e";
        Properties properties = new Properties();
        properties.put(PropertyKeyConst.SERVER_ADDR, serverAddr);
        properties.put(PropertyKeyConst.NAMESPACE, tenant);
        ConfigService configService = NacosFactory.createConfigService(properties);
        String content = configService.getConfig(dataId, group, 5000);
        System.out.println("配置为：" + content);
        configService.addListener(dataId, group, new Listener() {
            @Override
            public void receiveConfigInfo(String configInfo) {
                System.out.println("监听器-配置更新了：" + configInfo);
            }

            @Override
            public Executor getExecutor() {
                return null;
            }
        });

        boolean isPublishOk = configService.publishConfig(dataId, group, "application.name=nacos");
        System.out.println("发布了配置：" + isPublishOk);

        Thread.sleep(3000);
        content = configService.getConfig(dataId, group, 5000);
        System.out.println("配置为：" + content);

        Thread.sleep(3000);
        boolean isPublishOk1 = configService.publishConfig(dataId, group, "application.name=nacos1");
        System.out.println("更新了配置：" + isPublishOk1);

        Thread.sleep(3000);
        content = configService.getConfig(dataId, group, 5000);
        System.out.println("更新后配置为：" + content);

        Thread.sleep(3000);
        boolean isRemoveOk = configService.removeConfig(dataId, group);
        System.out.println("删除了配置：" + isRemoveOk);

        Thread.sleep(3000);
        content = configService.getConfig(dataId, group, 5000);
        System.out.println("删除后配置为：" + content);
        Thread.sleep(300000);
    }
}
```

#### 使用 MySQL 数据源
Nacos 默认使用嵌入式数据库存储配置信息，数据源支持换成 MySQL 。
首先使用 conf/nacos-mysql.sql 创建数据库。第二步将数据库配置加入 conf/application.properties 中。
```
spring.datasource.platform=mysql

db.num=1
db.url.0=jdbc:mysql://localhost:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true
db.user=root
db.password=123456
```

#### 管理多环境配置文件
通常会有 3 个环境 dev 、test 、prod ，三个环境的配置文件在 nacos 中进行管理。在项目中只存放 nacos config 相关的配置。通过 profiles 来区分不同环境的配置。
```
# dev 配置
spring:
  profiles: dev
nacos:
  config:
    server-addr: 127.0.0.1:8848
    namespace: af40020e-9c75-4174-be5b-796622544bae

---
#test 配置
spring:
  profiles: test
nacos:
  config:
    server-addr: 127.0.0.1:8848
    namespace: 33d746ec-529e-472c-a6d1-302ee741e6cf

---
# prod 配置
spring:
  profiles: prod
nacos:
  config:
    server-addr: 127.0.0.1:8848
    namespace: d8038098-05b5-4a9a-9d55-a274eecf1e3a
---


management:
  endpoints:
    web:
      exposure:
        include: '*'
  endpoint:
    health:
      show-details: always

spring:
  application:
    name: spring-boot-nacos-example

```
启动时通过 ```java -jar xxx.jar --spring.profiles.active=dev``` 来指定运行环境。