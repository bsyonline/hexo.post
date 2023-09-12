---
title: Build Ci/CD 
tags:
  - DevOps
category:
  - DevOps
author: bsyonline
lede: 没有摘要
date: 2023-05-31 23:20:09
thumbnail:
---



### centos

```text
yum install -y yum-utils device-mapper-persistent-data lvm2
```



```
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```



### docker

查看版本

```
yum list docker-ce --showduplicates | sort -r 
```

安装

```
yum install -y docker-ce-24.0.2-1.el7 docker-ce-cli-24.0.2-1.el7 containerd.io docker-buildx-plugin docker-compose-plugin
```

启动

```
systemctl start docker
```

开机启动

```
systemctl enable docker
```



```
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://xqnrfb7c.mirror.aliyuncs.com"],
  "dns" :[ "114.114.114.114", "8.8.8.8" ]
}
EOF
```



```
systemctl daemon-reload
systemctl restart docker
```



### jenkins

修改权限

```
chown -R 1000 /data/jenkins_home/
```

起镜像

```
docker run -di --name=jenkins -p 8080:8080 -v /data/jenkins_home:/var/jenkins_home jenkins/jenkins:lts
```

访问主页

```
http://192.168.93.201:8080/
```

从 docker logs 中获取管理员密码输入，然后安装插件，安装完进入主页。第一次登陆后记得改密码。

jenkins 的 lts 镜像的版本很低，很多插件无法安装，需要升级 jenkins 的版本。

下载 jenkins.war 挂载到容器中。

```
wget https://mirrors.jenkins.io/war-stable/latest/jenkins.war
```

进入容器。

```
docker exec -it -u root jenkins /bin/bash
```

替换 jenkins.war 

```
mv /usr/share/jenkins/jenkins.war /usr/share/jenkins/jenkins.war_bak
cp /var/jenkins_home/jenkins.war /usr/share/jenkins/jenkins.war
```

退出容器重启。

配置jdk和maven

下载 jdk 的包，解压到 /opt/java/jdk8 ，在全局工具配置>JDK>JDK 安装>新增 JDK>解压 \*.zip/\*.tar.gz 的解压目录中指定 /opt/java/jdk8 路径。

下载 maven 的包，解压到 /opt/apache-maven-3.9.2 ，在全局工具配置>Maven>Maven 安装>新增 安装>解压 \*.zip/\*.tar.gz 的解压目录中指定 /opt/apache-maven-3.9.2 路径。

配置ssh

安装 Publish Over SSH 插件。

系统管理>System>Publish over SSH>SSH Servers>新增，输入server地址、用户名、密码。

新建测试任务

新建任务>构建一个maven项目。

源码管理

设置 git 仓库，由于是 docker 部署的 jenkins ，所以凭据要用 docker 容器内生成的 ssh key 。分支根据实际选择。

Build>Goals and options

`clean install -Dmaven.test.skip=true`

Post Steps>Run only if build succeeds>Add post-build step>Send files or execute commands over SSH 

选择目标 server ，

Transfers>Transfer Set Source files 写要部署的文件，比如这里要部署一个 jar 包，build 之后的路径就是 target/jenkins-test-1.0-SNAPSHOT.jar 。

Remove prefix 写 target/

Remote directory 写要部署的远程服务器的路径，这个地方的路径是 ssh Servers 配置的 Remote Directory 的相对路径，我的 Remote Directory 是 /data ，这里写 workspace/jenkins-test ，实际在远程服务器上的部署路径就是 /data/workspace/ 。

Exec command 写需要执行的命令，比如启动 jar ，java -jar /data/workspace/jenkins-test/jenkins-test-1.0-SNAPSHOT.jar 

创建完就可以构建了。