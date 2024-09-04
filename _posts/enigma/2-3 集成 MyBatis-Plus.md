---
title: Java
tags: 
category: 
author: bsyonline
lede: 没有摘要
date: 2023-05-31 23:20:09
thumbnail:
---


上一节我们创建简单的 web 项目，这节我们来集成 orm 框架 MyBatis-Plus 。
首先我们需要引入 MyBatis-Plus 的依赖。根据官网说明加入依赖。

```xml
<dependency>  
    <groupId>com.baomidou</groupId>  
    <artifactId>mybatis-plus-spring-boot3-starter</artifactId>  
    <version>3.5.7</version>  
</dependency>
```

创建 dao 层 entity 和 mapper 。

```java
package com.enigma.backend.dao.entity;  
  
import com.baomidou.mybatisplus.annotation.*;  
import com.fasterxml.jackson.annotation.JsonProperty;  
import lombok.Getter;  
import lombok.Setter;  
  
import java.io.Serial;  
import java.io.Serializable;  
import java.time.LocalDateTime;  
  
@Getter  
@Setter  
@TableName("sys_user")  
public class SysUser implements Serializable {  
  
    @Serial  
    private static final long serialVersionUID = 1L;  
  
    @TableId(value = "id", type = IdType.AUTO)  
    private Integer id;  
  
    /**  
     * 用户名  
     */  
    private String username;  
  
    /**  
     * 昵称  
     */  
    private String nickName;  
  
    /**  
     * 头像  
     */  
    private String avatar;  
  
    /**  
     * 密码  
     */  
    private String password;  
  
    /**  
     * 邮箱  
     */  
    private String email;  
  
    /**  
     * 手机号  
     */  
    private String mobile;  
  
    /**  
     * 创建时间  
     */  
    @TableField(value = "created_at", fill = FieldFill.INSERT)  
    @JsonProperty("created_at")  
    private LocalDateTime createdAt;  
  
    /**  
     * 更新时间  
     */  
    @TableField(value = "updated_at", fill = FieldFill.UPDATE)  
    private LocalDateTime updatedAt;  
  
    /**  
     * 是否删除 -1：已删除   0：正常  
     */  
    @TableField("deleted_at")  
    private int deletedAt;  
}
```

```java
package com.enigma.backend.dao.mapper;  
  
import com.enigma.backend.dao.entity.SysUser;  
import com.baomidou.mybatisplus.core.mapper.BaseMapper;  
  
public interface SysUserMapper extends BaseMapper<SysUser> {  
  
}
```


增加 MapperScan 以便能够扫描到 mapper 类。

```java
package com.enigma.backend.config;  
  
import org.mybatis.spring.annotation.MapperScan;  
import org.springframework.context.annotation.Configuration;  
  
@Configuration  
@MapperScan("com.enigma.backend.dao.mapper")  
public class MyBatisPlusConfig {  
}
```

然后是 serveice 层。定义接口 ISysUserService 和 实现类 SysUserServiceImpl 。

```java
package com.enigma.backend.service;  
  
import com.enigma.backend.model.dto.UserDTO;  
  
import java.util.List;  
  
public interface ISysUserService {  
  
    UserDTO findById(Integer id);  
   
}
```

```java
package com.enigma.backend.service.impl;  
  
import com.enigma.backend.dao.entity.SysUser;  
import com.enigma.backend.dao.mapper.SysUserMapper;  
import com.enigma.backend.model.dto.UserDTO;  
import com.enigma.backend.service.ISysUserService;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.stereotype.Service;  
  
import java.util.List;  
  
@Service  
public class SysUserServiceImpl implements ISysUserService {  
  
    private SysUserMapper sysUserMapper;  
  
    @Autowired  
    public void setSysUserMapper(SysUserMapper sysUserMapper) {  
        this.sysUserMapper = sysUserMapper;  
    }  
  
    public UserDTO findById(Integer id) {  
        SysUser sysUser = sysUserMapper.selectById(id);
        UserDTO userDTO = new UserDTO(); 
        userDTO.setUsername(sysUser.getUsername());
        userDTO.setNickName(sysUser.getNickName());
        userDTO.setAvatar(sysUser.getAvatar());
        userDTO.setEmail(sysUser.getEmail());
        userDTO.setMobile(sysUser.getMobile());
        return userDTO;  
    }  

	@Override  
	public void save(UserDTO userDTO) {  
	    SysUser sysUser = new SysUser();  
		sysUser.setUsername(userDTO.getUsername());  
		sysUser.setPassword(userDTO.getPassword());  
		sysUser.setNickName(userDTO.getNickName());  
		sysUser.setAvatar(userDTO.getAvatar());  
		sysUser.setEmail(userDTO.getEmail());  
		sysUser.setMobile(userDTO.getMobile());  
		sysUserMapper.insert(sysUser);
	}
}
```

最后是 controller 。

```java
package com.enigma.backend.controller;  
  
import com.enigma.backend.model.dto.UserDTO;  
import com.enigma.backend.service.ISysUserService;  
import org.springframework.beans.factory.annotation.Autowired;  
import org.springframework.http.ResponseEntity;  
import org.springframework.web.bind.annotation.*;  
  
import java.util.List;  
  
@RestController  
@RequestMapping("/api/users")  
public class SysUserController {  

    ISysUserService sysUserService;  
  
    @Autowired  
    public void setSysUserService(ISysUserService sysUserService) {  
        this.sysUserService = sysUserService;  
    }  
  
    @GetMapping("{id}")  
    public ResponseEntity<UserDTO> findById(@PathVariable("id") Integer id) {  
        UserDTO userDTO = sysUserService.findById(id);  
        return ResponseEntity.ok(userDTO);  
    }  

	@PostMapping  
	public ResponseEntity<UserDTO> add(@RequestBody UserDTO userDTO) {  
	    sysUserService.save(userDTO);  
	    return ResponseEntity.ok(userDTO);  
	}
  
}
```

启动服务发送 GET localhost:8081/api/users/1 请求可看到结果。

此时，虽然已经能够插入用户数据，但是表里的 ``created_at`` 字段是空值，如果我们想让 MabatisPlus 自动填充，我们需要实现 ``MetaObjectHandler`` 接口。

```java
package com.enigma.backend.config;  
  
import com.baomidou.mybatisplus.core.handlers.MetaObjectHandler;  
import org.apache.ibatis.reflection.MetaObject;  
import org.springframework.stereotype.Component;  
  
import java.time.LocalDateTime;  
  
public class LocalDateTimeMetaObjectHandler implements MetaObjectHandler {  
    @Override  
    public void insertFill(MetaObject metaObject) {  
        this.strictInsertFill(metaObject, "createdAt", LocalDateTime.class, LocalDateTime.now());  
    }  
  
    @Override  
    public void updateFill(MetaObject metaObject) {  
        this.strictUpdateFill(metaObject, "updatedAt", LocalDateTime.class, LocalDateTime.now());  
    }  
}
```

同时还要在 ``@TableField`` 指定 fill 的时机，这样，对应所有的 createdAt 和 updatedAt 为空时都可以自动填充了。