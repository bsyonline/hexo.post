---
title: go
tags: 
category: []
author: bsyonline
lede: 没有摘要
date: 2023-05-31 23:20:09
thumbnail:
---


创建一个 gin web 项目 ``go mod init xxx`` 。规划一下目录结构。

```
|-cmd
  |-config.yaml        配置文件
  |-main.go            程序入口
 _pkg
  |-common             基础包
  |-controller         控制器
  |-dao                数据层
  |-logger             日志
  |-model              模型
    |-dto
    |-vo
  |-router             路由
  |-service            业务层
  |-settings           配置解析
  |-util               工具类
```

首先创建 cmd/main.go 。
```go
package main

import (  
    "github.com/gin-gonic/gin"
)
    
func main() {
	//初始化配置

    //初始化日志

    //初始化数据库

    //初始化路由

    //启动服务器
    r := gin.Default()
    r.GET("/test", func(c *gin.Context) {  
	    c.JSON(200, gin.H{  
	       "message": "success",  
	    })
	})
	err = r.Run()  
	if err != nil {  
		return  
	}
}
```

``go mod tidy`` 安装依赖，然后启动服务，访问 localhost:8080/test ，进行测试。