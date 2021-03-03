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

这样就实现了一个 Client 。

在我们自己的 endpoint 里就可以通过 OAuth2 认证拿到我们想要的信息了。

```java
@RequestMapping("/hello")
public String hello(@RegisteredOAuth2AuthorizedClient OAuth2AuthorizedClient authorizedClient, @AuthenticationPrincipal OAuth2User principal,
                        Model model) {
        log.info("username={}", principal.getName());
        log.info("username={}", SecurityContextHolder.getContext().getAuthentication().getName());
        log.info("attributes={}", principal.getAttributes());
        log.info("authorities={}", principal.getAuthorities());
        log.info("clientScopes={}", authorizedClient.getClientRegistration().getScopes());
        log.info("clientName={}", authorizedClient.getClientRegistration().getClientName());
        model.addAttribute("username", principal.getName());
        return "welcome";
}
```

