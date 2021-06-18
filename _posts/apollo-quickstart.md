---
title: Apollo Quickstart
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-05-15 13:09:37
thumbnail:
---



#### Apollo Client 使用

假设已在 Apollo 上创建好项目和配置：

​	AppId=App01

​	应用名称=apollo-example

​	配置项 key=timeout，value=100

##### 1. 加入依赖

```
<dependencies>
    <dependency>
        <groupId>com.ctrip.framework.apollo</groupId>
        <artifactId>apollo-core</artifactId>
        <version>1.8.0</version>
    </dependency>
    <dependency>
        <groupId>com.ctrip.framework.apollo</groupId>
        <artifactId>apollo-client</artifactId>
        <version>1.8.0</version>
    </dependency>
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.30</version>
    </dependency>
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>
</dependencies>
```

##### 2. 加入 Apollo 配置

在 resources/META-INF/app.properties 中加入 Apollo 配置。

```
app.id=App01
apollo.meta=http://hadoop1:8080
```

##### 3. 代码示例

3.1 主动获取

```java
public class GetApolloConfigTest {
    public static void main(String[] args) throws InterruptedException {
        Config appConfig = ConfigService.getAppConfig();
        while (true) {
            String value = appConfig.getProperty("timeout", "50");
            System.out.println("timeout=" + value);
            TimeUnit.SECONDS.sleep(1);
        }
    }
}
```

程序输出timeout=100，修改 Apollo 配置，timeout 改成 200 ，发布后程序输出变成 timeout=200 。

3.2 注册监听器

```java
public class GetApolloConfigTest1 {
    public static void main(String[] args) throws InterruptedException {
        Config config = ConfigService.getAppConfig(); //config instance is singleton for each namespace and is never null
        config.addChangeListener(new ConfigChangeListener() {
            @Override
            public void onChange(ConfigChangeEvent changeEvent) {
                System.out.println("Changes for namespace " + changeEvent.getNamespace());
                for (String key : changeEvent.changedKeys()) {
                    ConfigChange change = changeEvent.getChange(key);
                    System.out.println(String.format("Found change - key: %s, oldValue: %s, newValue: %s, changeType: %s", change.getPropertyName(), change.getOldValue(), change.getNewValue(), change.getChangeType()));
                }
            }
        });
        Thread.sleep(Integer.MAX_VALUE);
    }
}
```

修改 timeout 为 300 ，会输出

```
Changes for namespace application
Found change - key: timeout, oldValue: 200, newValue: 300, changeType: MODIFIED
```

#### 与 SpringBoot 集成