---
title: Nexus 伺服搭建
date: 2016-02-22 15:53:46
tags:
 - Git
 - Nexus
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---


### 下载

下载最新的 Nexus OSS

```
wget http://www.sonatype.org/downloads/nexus-latest-bundle.tar.gz
```

### 解压
```
tar -zxf nexus-latest-bundle.tar.gz  
mv nexus-2.11.1-01 nexus2

chown -R nexus2
chown -R sonatype-work
```
### 启动
```
cd nexus2
./bin/nexus start
```
### 安装服务

修改 ```bin/nexus``` 的 ```RUN_AS_USER=root```

创建服务

```
ln -s /usr/nexus2/bin/nexus /etc/init.d/nexus
source /etc/profile
```

启动服务

```
service nexus start
```

开机启动

```
chkconfig --add nexus  
chkconfig --levels 345 nexus on  
```
### 访问

在浏览器中输入 [http://192.168.1.201:8081/nexus/](http://192.168.1.201:8081/nexus/)
![img](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/9.png)

### 配置

管理员账号 admin ，默认密码 admin123 ，通过右上角登录。

### 添加代理仓库

点击左侧的 Repositories ，然后再点击右侧的 Add ，会弹出下拉菜单，选择 Proxy Repository ，如下配置。
![https://raw.githubusercontent.com/bsyonline/pic/master/10.png](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/10.png)
选中 Public Repositories ，将新建的 Proxy Repository 加入到 Public Repositories 组，并 Update Index 。
![https://raw.githubusercontent.com/bsyonline/pic/master/11.png](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/11.png)

### 配置 maven

修改 maven 的 **settings.xml** 文件
```
<?xml version="1.0" encoding="UTF-8"?>

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <!--设置本地仓库地址-->	  
    <localRepository>D:\mvn\repository</localRepository>
    <pluginGroups>
        <pluginGroup>org.mortbay.jetty</pluginGroup>
    </pluginGroups>
    <proxies>
    </proxies>
    <!--设置 Nexus 认证信息-->
    <servers>
        <server>
            <id>local-repo</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
    </servers>
    <!--设置 Nexus 镜像，只要本地没对应的，则到 Nexus 去找-->
    <mirrors>
        <!-- mirrorOf 会按照 id 匹配查找 -->
        <mirror>
            <id>local-repo</id>
            <mirrorOf>local-repo</mirrorOf>
            <url>http://192.168.0.201:8081/nexus/content/groups/public</url>
        </mirror>
        <mirror>
            <id>spring-milestone</id>
            <mirrorOf>spring-milestone</mirrorOf>
            <url>http://192.168.0.201:8081/nexus/content/repositories/spring-milestone/</url>
        </mirror>
        <mirror>
            <id>spring-snapshot</id>
            <mirrorOf>spring-snapshot</mirrorOf>
            <url>http://192.168.0.201:8081/nexus/content/repositories/spring-snapshot/</url>
        </mirror>
    </mirrors>
    <profiles>
        <profile>
            <id>local-repo</id>
            <properties>
                <downloadSources>true</downloadSources>
                <downloadJavadocs>true</downloadJavadocs>
            </properties>
            <!-- nexus 配置了三个 proxy 分别是 aliyun, spring-milestone, spring-snapshot -->
            <repositories>
                <repository>
                    <id>local-repo</id>
                    <name>Local Repository</name>
                    <url>http://192.168.0.201:8081/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>never</updatePolicy>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </repository>
                <repository>
                    <id>spring-milestone</id>
                    <name>Spring Milestone Repository</name>
                    <url>http://192.168.0.201:8081/nexus/content/repositories/spring-milestone/</url>
                    <releases>
                        <enabled>true</enabled>
                    </releases>
                    <snapshots>
                        <enabled>false</enabled>
                    </snapshots>
                    <layout>default</layout>
                </repository>
                <repository>
                    <id>spring-snapshot</id>
                    <name>Spring Snapshot Repository</name>
                    <url>http://192.168.0.201:8081/nexus/content/repositories/spring-snapshot/</url>
                    <releases>
                        <enabled>false</enabled>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                    </snapshots>
                    <layout>default</layout>
                </repository>
            </repositories>
            <pluginRepositories>
                <pluginRepository>
                    <id>local-repo</id>
                    <url>http://192.168.0.201:8081/nexus/content/groups/public</url>
                    <releases>
                        <enabled>true</enabled>
                        <updatePolicy>daily</updatePolicy>
                    </releases>
                    <snapshots>
                        <enabled>true</enabled>
                        <updatePolicy>always</updatePolicy>
                    </snapshots>
                </pluginRepository>
            </pluginRepositories>
        </profile>
        <!-- 代码检查 -->
        <profile>
            <id>sonar</id>
            <activation>
                <activeByDefault>false</activeByDefault>
            </activation>
            <properties>
                <sonar.jdbc.url>jdbc:mysql://192.168.0.201:3306/sonar?useUnicode=true&amp;characterEncoding=utf8&amp;rewriteBatchedStatements=true</sonar.jdbc.url>
                <sonar.jdbc.driver>com.mysql.jdbc.Driver</sonar.jdbc.driver>
                <sonar.jdbc.username>sonar</sonar.jdbc.username>
                <sonar.jdbc.password>sonar</sonar.jdbc.password>
                <sonar.host.url>http://192.168.0.201:9000</sonar.host.url>
            </properties>
        </profile>
    </profiles>
    <activeProfiles>
        <activeProfile>local-repo</activeProfile>
    </activeProfiles>
</settings>
```

第一次安装后，nexus 伺服仓库是空的，通过maven下载jar包显示从伺服仓库下载，实际还是从公网下载，然后保存到本地，以后再下载相同的 jar 包就直接从伺服仓库下载了。
