---
title: Getting Start with Spring Cloud Dataflow
tags:
  - Spring Cloud
  - Microservices
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-10-19 11:58:49
thumbnail:
---


### **配置环境**

1. 下载 spring cloud dataflow 的 jar 包

   ```
   #spring-cloud-dataflow-server-local
   wget https://repo.spring.io/release/org/springframework/cloud/spring-cloud-dataflow-server-local/1.6.3.RELEASE/spring-cloud-dataflow-server-local-1.6.3.RELEASE.jar
   
   #spring-cloud-dataflow-shell
   wget https://repo.spring.io/release/org/springframework/cloud/spring-cloud-dataflow-shell/1.6.3.RELEASE/spring-cloud-dataflow-shell-1.6.3.RELEASE.jar
   ```

2. 启动服务

   启动 rabbitmq

   ```
   docker run --name dataflow-rabbit -p 15672:15672 -p 5672:5672 -d rabbitmq:3.7-management
   ```

   启动 redis

   ```
   docker run --name dataflow-redis -p 6379:6379 -d redis:3.0
   ```

   启动 mysql

   ```
   docker run --name mysql -e MYSQL_ROOT_PASSWORD=123456 -e MYSQL_DATABASE=scdf -p 3306:3306 -d mysql:5.7
   ```

   启动 spring-cloud-dataflow-server-local

   ```
   java -jar spring-cloud-dataflow-server-local-1.6.3.RELEASE.jar --spring.datasource.url=jdbc:mysql://localhost:3306/scdf --spring.datasource.username=root --spring.datasource.password=123456 --spring.datasource.driver-class-name=org.mariadb.jdbc.Driver --spring.rabbitmq.host=127.0.0.1 --spring.rabbitmq.port=5672 --spring.rabbitmq.username=guest --spring.rabbitmq.password=guest
   ```

3. 启动 spring-cloud-dataflow-shell

   ```
   $ java -jar spring-cloud-dataflow-shell-1.6.3.RELEASE.jar 
   ____                              ____ _                __
   / ___| _ __  _ __(_)_ __   __ _   / ___| | ___  _   _  __| |
   \___ \| '_ \| '__| | '_ \ / _` | | |   | |/ _ \| | | |/ _` |
   ___) | |_) | |  | | | | | (_| | | |___| | (_) | |_| | (_| |
   |____/| .__/|_|  |_|_| |_|\__, |  \____|_|\___/ \__,_|\__,_|
   ____ |_|    _          __|___/                 __________
   |  _ \  __ _| |_ __ _  |  ___| | _____      __  \ \ \ \ \ \
   | | | |/ _` | __/ _` | | |_  | |/ _ \ \ /\ / /   \ \ \ \ \ \
   | |_| | (_| | || (_| | |  _| | | (_) \ V  V /    / / / / / /
   |____/ \__,_|\__\__,_| |_|   |_|\___/ \_/\_/    /_/_/_/_/_/
   
   1.6.3.RELEASE
   
   Welcome to the Spring Cloud Data Flow shell. For assistance hit TAB or type "help".
   dataflow:>
   ```

   如果出现 ```dataflow-unknown:>``` 需要设置服务器

   ```server-unknown:&gt;dataflow config server http://localhost:9393
   server-unknown:>dataflow config server http://localhost:9393
   Shell mode: classic, Server mode: classic
   dataflow:>
   ```


### 使用界面配置 UpperCaseStream

启动 server 后访问 dashboard [http://localhost:9393/dashboard](http://localhost:9393/dashboard) 。这时 dashboard 里还没什么内容，spring 提供了一些可用的 app ，我们需要先导入这些 app 。

可以使用 http 方式导入 [bit.ly/Celsius-SR3-stream-applications-rabbit-maven](http://bit.ly/Celsius-SR3-stream-applications-rabbit-maven) 。

![mark](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/131648605.png)

也可以使用文件导入。

![mark](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/131813750.png)

导入成功后就可以看到下面的内容。

![mark](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/131904382.png)

接下来就使用导入的 3 个 App 来完成第一个例子。

1. 先创建一个 stream

   ![mark](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/132043862.png)

2. 将 http ，transform ，file 三个组件拖拽到面板

   ![mark](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/133611052.png)

3. 配置 file 的信息

   ![](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/125512120.png)

4. 配置 transform 的信息

   ![](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/125302026.png)

5. 配置 file 的信息

   ![](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/125349123.png)

6. 将 app 按顺序连接起来

   ![](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/125604831.png)

7. 最后创建并部署

   ![](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/125711260.png)

8. 部署完成在 Runtime 中可以看到 stream 的信息

   ![mark](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/131950197.png)

9. 发送一个 http 请求，然后查看文件可看到结果

   ```
   dataflow:>http post --target http://10.0.75.1:61068 --data "hello world"
   > POST (text/plain) http://10.0.75.1:61068 hello world
   > 202 ACCEPTED
   ```

![mark](http://pgu17t3sd.bkt.clouddn.com/pic/20181021/131010335.png)

### **官方文档上的 httptest**

上边我们通过 dashboard 部署了一个 stream，接下来我们通过官方文档的 httptest 来看看如何使用 cli 部署 stream 。

1. 创建 stream

   ```
   stream create --name httptest --definition "http | log" --deploy
   ```

2. 查看 stream

   ```
   dataflow:>stream list
   ╔═══════════╤═════════════════╤═════════════════════════════════════════╗
   ║Stream Name│Stream Definition│                 Status                  ║
   ╠═══════════╪═════════════════╪═════════════════════════════════════════╣
   ║httptest   │http | log       │The stream has been successfully deployed║
   ╚═══════════╧═════════════════╧═════════════════════════════════════════╝
   ```

3. 查看运行的 App 信息

   ```
   dataflow:>runtime apps 
   ╔════════════════════╤═══════════╤════════════════════════════════════════════════════════════════════════════════════════════════════════════╗
   ║App Id / Instance Id│Unit Status│                                       No. of Instances / Attributes                                        ║
   ╠════════════════════╪═══════════╪════════════════════════════════════════════════════════════════════════════════════════════════════════════╣
   ║httptest.http       │ deployed  │                                                     1                                                      ║
   ╟┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┼┈┈┈┈┈┈┈┈┈┈┈┼┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈╢
   ║                    │           │       guid = 54544                                                                                         ║
   ║                    │           │        pid = 2424                                                                                          ║
   ║                    │           │       port = 54544                                                                                         ║
   ║httptest.http-0     │ deployed  │     stderr = /tmp/spring-cloud-deployer-69437306715667832/httptest-1539933007655/httptest.http/stderr_0.log║
   ║                    │           │     stdout = /tmp/spring-cloud-deployer-69437306715667832/httptest-1539933007655/httptest.http/stdout_0.log║
   ║                    │           │        url = http://127.0.0.1:54544                                                                        ║
   ║                    │           │working.dir = /tmp/spring-cloud-deployer-69437306715667832/httptest-1539933007655/httptest.http             ║
   ╟────────────────────┼───────────┼────────────────────────────────────────────────────────────────────────────────────────────────────────────╢
   ║httptest.log        │ deployed  │                                                     1                                                      ║
   ╟┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┼┈┈┈┈┈┈┈┈┈┈┈┼┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈┈╢
   ║                    │           │       guid = 17251                                                                                         ║
   ║                    │           │        pid = 2406                                                                                          ║
   ║                    │           │       port = 17251                                                                                         ║
   ║httptest.log-0      │ deployed  │     stderr = /tmp/spring-cloud-deployer-69437306715667832/httptest-1539933007348/httptest.log/stderr_0.log ║
   ║                    │           │     stdout = /tmp/spring-cloud-deployer-69437306715667832/httptest-1539933007348/httptest.log/stdout_0.log ║
   ║                    │           │        url = http://127.0.0.1:17251                                                                        ║
   ║                    │           │working.dir = /tmp/spring-cloud-deployer-69437306715667832/httptest-1539933007348/httptest.log              ║
   ╚════════════════════╧═══════════╧════════════════════════════════════════════════════════════════════════════════════════════════════════════╝
   ```

4. 发一条测试数据

   ```
   dataflow:>http post --target http://127.0.0.1:54544 --data "hello world"
   > POST (text/plain) http://127.0.0.1:54544 hello world
   > 202 ACCEPTED
   ```

5. 在 ```/tmp/spring-cloud-deployer-69437306715667832/httptest-1539933007348/httptest.log/stdout_0.log``` 中可以看到打印的 hello world 信息。





