配置文件写法：

```yml
spring:
  security:
    oauth2:
      client:
        registration:
          github:
            client-id: b2cedfecec7b6dd2d594
            client-secret: b9d75d571eb6e884b5873d627462e380e86d4f6a
          alphax:
            client-id: 1a6642b4-622d-11eb-a707-00059a3c7a00
            client-secret: 123
            authorization-grant-type: authorization_code
            redirect-uri: http://localhost:8080/oauth/code/alphax
            provider:
              alphax:
                authorization-uri: http://localhost:8082/oauth/authorize
                token-uri: http://localhost:8082/oauth/token
                user-info-uri: http://localhost:8081/users/info
                user-name-attribute: name
```

spring boot 会将 spring.security.oauth2.client.registration.alphax 下的属性绑定到 ClientRegistration 实例。