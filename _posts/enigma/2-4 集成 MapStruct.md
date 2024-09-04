---
title: Java
tags: 
category: 
author: bsyonline
lede: 没有摘要
date: 2023-05-31 23:20:09
thumbnail:
---


上一节我们集成了 MyBatis-Plus ，并实现了一个 RESTFUL 接口 。我们的项目分为三层，数据层-服务层-控制层。层与层之间涉及到数据传输。可以看到我们定义了 2 个对象：Entity 对象和 DTO 对象。Entity 对象是和数据库表的映射，并不需要将所有的字段都暴露出去，通过 DTO 对象的转换，可以有选择的对外暴露字段，能够增加安全性。但是我们也看到在 2 个对象在服务层进行了转换，将 Entity 对象的属性通过 setter 复制给 DTO 对象。如果属性很多，一个个复制非常不方便，所以我们需要一些对象拷贝工具来简化我们的工作。
对象拷贝工具有很多，比如 Spring 的 BeanUtils ，但是它的性能不好。这里我们使用一种高性能的类库 MapStruct 。

##### MapStruct的关键特性

1. **类型安全**：MapStruct 在编译时检查映射规则，确保源对象和目标对象之间的属性映射是类型安全的。这减少了运行时因类型转换错误而导致的问题。
2. **性能**：生成的映射代码使用简单的 getters 和 setters ，避免了使用反射，因此在运行时可以提供更好的性能。
3. **易于理解和使用**：MapStruct 生成的代码简单易懂，开发者可以轻松阅读和理解映射逻辑。
4. **自定义映射**：MapStruct 允许开发者定义复杂的映射规则，包括深拷贝和自定义转换函数。
5. **错误提前暴露**：编译时就能发现潜在的错误，如映射不完整或映射方法不正确，这样可以提前修复问题，避免在运行时出现故障。

##### MapStruct的工作原理

MapStruct 基于 Java 的 JSR 269 规范，该规范允许在编译期处理注解。MapStruct 通过定义的注解处理器，在编译期读取映射接口，并生成相应的实现类。这个过程中，它会解析接口中声明的映射方法，并创建对应的 getters 和 setters 调用。

我们来看看如何将 MapStruct 集成到 springboot 的项目中并使用它。
首先添加 MapStruct 的依赖。

```xml
<dependency>  
    <groupId>org.mapstruct</groupId>  
    <artifactId>mapstruct</artifactId>  
    <version>1.5.5.Final</version>  
</dependency>
```

接下来定义 MapStruct 的接口并使用 @Mapper 注解 。

```java
package com.enigma.backend.model.dto;  
  
import com.enigma.backend.dao.entity.SysUser;  
import org.mapstruct.Mapper;  
  
import java.util.List;  
  
@Mapper(componentModel = "spring")  
public interface UserDTOMapper {  
  
    SysUser toEntity(UserDTO userDTO);  
  
    UserDTO fromEntity(SysUser sysUser);  
  
    List<UserDTO> fromEntity(List<SysUser> students);  
}
```

这样编译项目时，MapStruct 注解处理器会根据定义的映射规则生成实现类。我们在 service 层就可以通过注入或者 Mappers.getMapper() 来使用了。

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
    private UserDTOMapper userDTOMapper;
  
    @Autowired  
    public void setSysUserMapper(SysUserMapper sysUserMapper) {  
        this.sysUserMapper = sysUserMapper;  
    } 

	@Autowired  
	public void setUserDTOMapper(UserDTOMapper userDTOMapper) {  
		this.userDTOMapper = userDTOMapper;  
	}
  
    public UserDTO findById(Integer id) {  
        SysUser sysUser = sysUserMapper.selectById(id);
        return userDTOMapper.fromEntity(sysUser); 
    }  

}
```

这样我们就不用再挨个属性 set 了 。

到这里还有一个小坑，由于我们项目使用了 Lombok ，它也是编译的时候生成代码，所以和 MapStruct 一起使用的时候运行时会报找不到属性。我们需要通过 plugins 来解决这个问题。

在 maven-compiler-plugin 的 annotationProcessorPaths 中加入 lombok-mapstruct-binding 。

```xml
<plugin>  
    <groupId>org.apache.maven.plugins</groupId>  
    <artifactId>maven-compiler-plugin</artifactId>  
    <version>3.11.0</version>  
    <configuration>        
	    <annotationProcessorPaths>  
            <path>  
                <groupId>org.projectlombok</groupId>  
                <artifactId>lombok</artifactId>  
                <version>1.18.34</version>  
            </path>  
            <dependency>                
	            <groupId>org.projectlombok</groupId>  
                <artifactId>lombok-mapstruct-binding</artifactId>  
                <version>0.2.0</version>  
            </dependency>  
            <path>                
	            <groupId>org.mapstruct</groupId>  
                <artifactId>mapstruct-processor</artifactId>  
                <version>1.5.5.Final</version>  
            </path>  
        </annotationProcessorPaths>  
    </configuration>  
</plugin>
```

注意 lombok 和 mapstruct-processor 的顺序。这样就可以正常使用了。