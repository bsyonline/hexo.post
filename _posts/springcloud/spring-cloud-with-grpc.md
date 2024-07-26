---
title: Spring Cloud with gRPC
tags:
  - Microservices
  - Spring Cloud
  - gRPC
category:
  - Spring Cloud
author: bsyonline
lede: 没有摘要
date: 2018-06-09 16:12:25
thumbnail:
---

微服务架构中服务之间通信有很多种方式，大多数是使用 REST 。由于 rpc 在性能上的优势，使用 rpc 进行内部服务之间通信越来越普遍。rpc 框架有很多，gRPC 在性能、跨平台等方面的优秀表现，被很多公司作为选型方案。作为主流的微服务开发工具 Spring Cloud 对 gRPC 也提供了集成。

使用 gRPC 的 Spring Cloud 启动器 [https://github.com/yidongnan/grpc-spring-boot-starter](https://github.com/yidongnan/grpc-spring-boot-starter) 可以在 Spring Cloud 中很简单的集成 gRPC 。
#### **gRPC proto**
##### 1. 编写 proto
```
syntax = "proto3";

option java_multiple_files = true;
option java_package = "com.rolex.microservices.grpc";
option java_outer_classname = "ProductProto";
option objc_class_prefix = "HLW";

service ProductService {
  rpc FindProductByClientId (ProductRequest) returns (ProductResponse) {}
}

message ProductRequest {
  string user_id = 1;
}

message ProductResponse {
  uint64 product_id = 1;
  string user_id = 2;
  string client_id = 3;
}
``` 
##### 2. 添加 maven 配置
```
<dependencies>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-netty</artifactId>
            <version>${grpc.version}</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-protobuf</artifactId>
            <version>${grpc.version}</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-stub</artifactId>
            <version>${grpc.version}</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-alts</artifactId>
            <version>${grpc.version}</version>
        </dependency>
        <dependency>
            <groupId>io.grpc</groupId>
            <artifactId>grpc-testing</artifactId>
            <version>${grpc.version}</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-tcnative-boringssl-static</artifactId>
            <version>${netty.tcnative.version}</version>
        </dependency>
        <dependency>
            <groupId>com.google.api.grpc</groupId>
            <artifactId>proto-google-common-protos</artifactId>
            <version>1.0.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>2.18.3</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
    <build>
        <extensions>
            <extension>
                <groupId>kr.motd.maven</groupId>
                <artifactId>os-maven-plugin</artifactId>
                <version>1.5.0.Final</version>
            </extension>
        </extensions>
        <plugins>
            <plugin>
                <groupId>org.xolstice.maven.plugins</groupId>
                <artifactId>protobuf-maven-plugin</artifactId>
                <version>0.5.1</version>
                <configuration>
                    <protocArtifact>com.google.protobuf:protoc:3.5.1-1:exe:${os.detected.classifier}</protocArtifact>
                    <pluginId>grpc-java</pluginId>
                    <pluginArtifact>io.grpc:protoc-gen-grpc-java:${grpc.version}:exe:${os.detected.classifier}
                    </pluginArtifact>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <goal>compile</goal>
                            <goal>compile-custom</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>
        </plugins>
    </build>
```
#### **gRPC Server**
##### 1. 服务端代码
```
@GrpcService(ProductServiceGrpc.class)
public class ProductGrpcServerService extends ProductServiceGrpc.ProductServiceImplBase {
    
    @Override
    public void findProductByClientId(ProductRequest request, StreamObserver<ProductResponse> responseObserver) {
        ProductResponse productResponse = null;
        if ("1".equals(request.getUserId())) {
            productResponse = ProductResponse.newBuilder()
                .setClientId("1")
                .setUserId("1")
                .setClientId("1")
                .setProductId(1L)
                .build();
        }
        responseObserver.onNext(productResponse);
        responseObserver.onCompleted();
    }
    
}
```
##### 2. 配置文件添加 gRPC Server 配置
```
grpc:
  server:
    port: 9402
    host: ${spring.application.name}
```
#### **gRPC Client**
##### 1. 客户端代码
```
@Service
public class ProductService {

    @GrpcClient("product-center")
    Channel serverChannel;
    public ProductResponse findProductByUserId(String userId) {
        ProductServiceGrpc.ProductServiceBlockingStub stub = ProductServiceGrpc.newBlockingStub(serverChannel);
        ProductResponse response = stub.findProductByClientId(ProductRequest.newBuilder().setUserId(userId).build());
        return response;
    }

}
```
##### 2. 添加 gRPC Client 配置
```
grpc:
  client:
    product-center:
      port: 9402
```
