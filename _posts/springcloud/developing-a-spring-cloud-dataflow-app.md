---
title: Developing a Spring Cloud Dataflow App
tags:
  - Spring Cloud
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-10-19 15:47:40
thumbnail:
---



本节我们通过写程序完成一个小功能来了解如何开发 spring cloud dataflow app 。

方便起见，我们将通过 http 发送一个字符串，然后将字符串拆分并输出到文件。我们可以写一个 transform 来完成字符串拆分。

1. 首先创建一个 spring boot 程序并引入 spring-cloud-stream 的依赖

   ```
   <dependencies>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-stream-rabbit</artifactId>
       </dependency>
       <dependency>
           <groupId>org.apache.commons</groupId>
           <artifactId>commons-lang3</artifactId>
           <version>${commons-lang3.version}</version>
       </dependency>
       <dependency>
           <groupId>org.springframework.boot</groupId>
           <artifactId>spring-boot-starter-test</artifactId>
           <scope>test</scope>
       </dependency>
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-stream-test-support</artifactId>
           <scope>test</scope>
       </dependency>
   </dependencies>
   
   <build>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
       </plugins>
   </build>
   
   <distributionManagement>
       <repository>
           <id>mylocal</id>
           <name>Local Repository</name>
           <url>http://localhost:8081/nexus/content/repositories/releases</url>
       </repository>
       <snapshotRepository>
           <id>mylocal</id>
           <name>Local Repository</name>
           <url>http://localhost:8081/nexus/content/repositories/snapshots</url>
       </snapshotRepository>
   </distributionManagement>
   ```

   为了能将 jar 上传到 maven 仓库，我们加入了仓库的配置。

2. 加一些注解

   ```
   import org.apache.commons.lang3.StringUtils;
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.cloud.stream.annotation.EnableBinding;
   import org.springframework.cloud.stream.annotation.StreamListener;
   import org.springframework.cloud.stream.messaging.Processor;
   import org.springframework.cloud.stream.messaging.Sink;
   import org.springframework.messaging.handler.annotation.SendTo;
   
   import java.util.Arrays;
   
   @EnableBinding(Processor.class)
   @SpringBootApplication
   public class ProcessorApplication {
       
       public static void main(String[] args) {
           SpringApplication.run(ProcessorApplication.class, args);
       }
       
       @StreamListener(target = Sink.INPUT)
       @SendTo(Processor.OUTPUT)
       public String split(String s) {
           return StringUtils.join(s.split(","), " ");
       }
   }
   ```

   使用两个注解，除此之外不需要其他配置，spring cloud dataflow 会自动完成配置。

3. 注册 app

   ```
   dataflow:>app register --name split-processor --type processor --uri maven://com.rolex.microlabs:transform-app:1.0-SNAPSHOT
   ```

   uri 的格式为：```maven://groupId:artifactId:version```

4. 创建和部署

   注册成功后就可以使用 app 来完成 stream 创建了。

   ```
   stream create --name splittest --definition "splitInput: http | splitTransform: split-processor | splitOutput: file --directory=/tmp --name=split.txt" --deploy true
   ```

   这时你会看到一个这样的错误：

   ```
   Command failed org.springframework.cloud.dataflow.rest.client.DataFlowClientException: Failed to resolve MavenResource: com.rolex.microlabs:transform-app:jar:1.0-SNAPSHOT. Configured remote repositories: : [springRepo],[repo1]
   ```

   很简单 spring cloud dataflow server 找不到我们写的 trasnform 。因为我们的程序并没有在 maven 的中央仓库中，所以我们需要使用私有 maven 仓库，让 spring cloud dataflow server 从私有仓库中下载 transform 。

   停止 spring cloud dataflow server ，加入 maven 仓库选项：

   ```
   --maven.localRepository=mylocal --maven.remote-repositories.repo1.url=http://localhost:8081/nexus/content/repositories/snapshots/
   ```

   然后重新启动。我们将 transform 上传到 maven 仓库之后再重新部署就可以正常部署了。

5. 部署成功后发一条测试数据

   ```
   dataflow:>http post --target http://127.0.0.1:46858 --data "spring,cloud,dataflow"
   > POST (text/plain) http://127.0.0.1:46858 spring,cloud,dataflow
   > 202 ACCEPTED
   ```

   然后查看 /tmp/split.txt 就可以看到 spring cloud dataflow 的信息了。



