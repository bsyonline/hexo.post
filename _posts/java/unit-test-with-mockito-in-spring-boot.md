---
title: Unit Test with Mockito in Spring Boot
tags:
  - Spring Boot
  - Mock
category:
  - Spring Boot
author: bsyonline
lede: 没有摘要
date: 2018-01-15 10:35:30
thumbnail:
---

在单元测试中，经常会遇到测试某一个方法，但是代码关联了很多模块，如 HttpServlet、数据库连接等，还可能关联其他的环境和服务，造成单元测试难度大，测试过程复杂。这个时候用 Mock 就能极大简化测试过程。Mock 会帮我们将单元测试依赖解耦，借助 Mock 模拟这些依赖，并验证所调用的依赖的行为。

Mock 的测试框架很多，Spring boot 中集成了 Mockito 。下面就来看看如何在 spring boot 中使用 mock 来做单元测试。

假如我们开发了一个查询接口，需要进行单元测试。

```
@Mapper
@Component
public interface UserDao {
    User findById(@Param("id") int id);
}
```

```
@Service
public class UserService {
    @Autowired
    UserDao userDao;
    public User findById(int id){
        return userDao.findById(id);
    }
}
```

```
@RestController
public class UserController {
    @Autowired
    UserService userService;
    @RequestMapping("/users/{id}")
    public User list(@PathVariable("id") int  id) {
        return userService.findById(id);
    }
}
```

如果不使用 Mock ，我们需要连接数据库和启动服务器。如果每次修改代码测试都要保证数据库和服务器环境正常，会使单元测试变得复杂且效率低下。所以我们可以使用 Mock 来模拟依赖对象进行测试。

```
import com.rolex.bean.User;
import com.rolex.controller.UserController;
import com.rolex.service.UserService;
import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;

import static org.mockito.Mockito.when;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;


@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootTestApplicationTests {
    
    MockMvc mockMvc;
    @Autowired
    UserController userController;
    @MockBean
    UserService userService;

    @Before
    public void setUp() {
        mockMvc = MockMvcBuilders.standaloneSetup(userController).build();
    }

    @Test
    public void findById() throws Exception {
        User user = new User("Phillip", 20);
        when(userService.findById(1)).thenReturn(user);
        Assert.assertEquals("Phillip", userService.findById(1).getName());
    }

    @Test
    public void findById1() throws Exception {
        User user = new User("Phillip", 20);
        when(userService.findById(1)).thenReturn(user);
        mockMvc.perform(MockMvcRequestBuilders.get("/users/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("Phillip"));
    }
}
```

完整代码参考：[https://github.com/bsyonline/spring-boot-test](https://github.com/bsyonline/spring-boot-test)



