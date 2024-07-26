---
title: Reference to OAuth2.0
tags:
  - untag
category:
  - uncategory
author: bsyonline
lede: 没有摘要
date: 2021-01-30 20:25:25
thumbnail:
---

#### 什么是 OAuth2

OAuth2 是一个授权框架，它可以让用户授权第三方程序使用服务提供方的用户数据，而不用将用户账户密码告诉第三方程序。

OAuth 有4种模式：

1. 授权码模式（authorization code）
2. 简化模式（implicit）
3. 密码模式（resource owner password credentials）
4. 客户端模式（client credentials）

授权码模式是最为完善和严谨的一种授权模式，网络上第三方认证大多使用的都是这种授权模式。

#### 授权码模式

授权码模式的流程：

1. 用户访问第三方应用；

2. 第三方应用向认证服务器发送请求获取授权码；

   这一步第三方应用需要向认证服务器发送请求，参数需要标识第三方应用的id和授权类型，比如：

   ```
   GET /ouath/authorize?response_type=code&client_id=1a6642b4-622d-11eb-a707-00059a3c7a00&state=xyz&redirect_uri=http://localhost:8081/callback
   ```

   - response_type：表示授权类型，必选项，此处的值固定为"code"
   - client_id：表示客户端的ID，第三方应该提前在认证服务器注册的 ID，必选项
   - redirect_uri：表示重定向URI，可选项
   - scope：表示申请的权限范围，可选项
   - state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

3. 认证服务器将浏览器页面重定向到登录授权页面，让用户选择是否授权；

4. 用户授权；

5. 认证服务器将授权码通过第三方应用的回调地址发送给第三方应用；

   ```
   HTTP/1.1 302 Found
   Location: http://localhost:8081/callback?code=4t7EFX&state=xyz
   ```

   这个回调地址是第三方应用提供的，用于接收授权码。

6. 第三方应用收到授权码后在使用授权码向认证服务器申请 token ；

   ```
   POST /oauth/token 
   
   grant_type=authorization_code&code=4t7EFX&client_id=1a6642b4-622d-11eb-a707-00059a3c7a00&client_secret=xxx
   ```

   - response_type：表示授权类型，必选项，此处的值固定为"code"
   - client_id：表示客户端的ID，必选项
   - client_secret：表示客户端的密码，必选项
   - scope：表示申请的权限范围，可选项

   一个授权码只能使用一次。

7. 认证服务器将 token 发送给第三方应用；

   ```json
   {
       "access_token": "86772cc2-e98f-4ea4-9738-31d620f06730",
       "token_type": "bearer",
       "refresh_token": "1abf9b64-803f-483e-8c3e-aa41b98b4b59",
       "expires_in": 16369,
       "scope": "read write"
   }
   ```

8. 第三方应用向资源服务器请求用户数据；

   ```
   /orders/list?access_token=86772cc2-e98f-4ea4-9738-31d620f06730
   ```

9. 资源服务器向认证服务器获取 token ；

10. 认证服务器将 token 发给资源服务器；

11. 资源服务器校验 token ；

12. 验证通过后，将用户数据发送给第三方应用。

#### 实现一个 OAuth2 Demo 

实现一个 OAuth2 的 Demo 可以按照 OAuth2 的规范自己来写 endpoint 的处理逻辑，我们这里主要介绍如何使用 spring security 来实现 OAuth2 。目前 spring security 实现 OAuth 主要有两种方式，1种是基于 spring social 框架，另一种是使用 spring security 5 。spring social 框架目前处于维护阶段，不再更新，spring security OAuth2 的框架在 spring security 5 之后进行了重写，所以我们这里介绍的是基于 spring security 5 框架实现 OAuth2 。

##### OAuth2 Server

OAuth2 包含 3 个角色，首先我们需要有一个授权服务器，比如使用 Github 的授权服务，那么我们首先需要 Github 上添加一个 app 的注册信息 [Add a new GitHub app](https://github.com/settings/developers) 。如果我们想自己来管理登录、登出认证及授权，我们也可以自己实现一个授权服务器。

##### OAuth2 Resource Server

OAuth2 的另一个角色是资源服务器，资源服务器主要提供 info 信息，在认证通过后，通过资源服务获取授权使用的资源。比如通过一个 endpoint 返回用户名字。

```java
@GetMapping("/users/info")
public SysUser info() {
    SysUser sysUser = new SysUser();
    sysUser.setUsername("John");
    return sysUser;
}
```



##### OAuth2 Client

spring security 5 提供了 OAuth Client 的支持，并且内置了 Github、Google、Feacbook、OKTA 的默认支持，如果使用这几种作为认证服务器，那么只需要配置 client-id 和 client-sercet 即可实现一个 OAuth Client 。

```yml
spring:
  security:
    oauth2:
      client:
        registration:
          githab:
            client-id: 1a6642b4-622d-11eb-a707-00059a3c7a00
            client-secret: 2b4-622d-11eb-a707-00059a3c7a
```

当然也支持使用其他的授权服务。我们需要自己添加 registration 和 provider 的信息。

```yml
spring:
  security:
    oauth2:
      client:
        registration:
          alphax:
            client-id: 1a6642b4-622d-11eb-a707-00059a3c7a00
            client-secret: 123
            client-authentiacation-method: basic
            authorization-grant-type: authorization_code
            scope: read,write
            redirect-uri: "{baseUrl}/login/oauth2/code/{registrationId}"
            provider: alphax
            client-name: AlphaX
        provider:
          alphax:
            authorization-uri: http://auth-server.com:8082/oauth/authorize
            token-uri: http://auth-server.com:8082/oauth/token
            user-info-uri: http://resource-server.com:8081/users/info
            user-name-attribute: username
```

