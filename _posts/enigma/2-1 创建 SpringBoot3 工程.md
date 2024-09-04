---
title: Java
tags: 
category: 
author: bsyonline
lede: 没有摘要
date: 2023-05-31 23:20:09
thumbnail:
---


从 start.spring.io 创建项目，选择需要的依赖。

在配置文件中增加数据库的连接信息，项目就可以启动起来了。

```yaml
spring:  
  application:  
    name: enigma-backend  
  datasource:  
    url: jdbc:mysql://127.0.0.1:3306/enigma_backend?useUnicode=true&characterEncoding=UTF-8&serverTimezone=GMT%2b8&allowMultiQueries=true  
    username: root  
    password: 123456  
    driver-class-name: com.mysql.cj.jdbc.Driver  
server:  
  port: 8081
```

然后我们简单写一个接口测试一下。

```java
import com.enigma.backend.model.dto.UserDTO;  
import com.enigma.backend.model.vo.LoginRequest;    
import org.springframework.http.ResponseEntity;  
import org.springframework.web.bind.annotation.PostMapping;  
import org.springframework.web.bind.annotation.RequestBody;  
import org.springframework.web.bind.annotation.RestController;  
  
@RestController  
public class LoginController {  
  
    @PostMapping("/login")  
    public ResponseEntity<UserDTO> login(@RequestBody LoginRequest loginRequest) { 
        if("admin".equals(loginRequest.getUsername())) {  
            UserDTO userDTO = new UserDTO();  
            userDTO.setUsername(loginRequest.getUsername());  
            return ResponseEntity.ok(userDTO);  
        } else {  
            return ResponseEntity.status(401).build();  
        }  
    }  
  
}
```

访问 POST localhost:8081/login 就可以完成简单的调用了。