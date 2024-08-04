---
title: go
tags: 
category: []
author: bsyonline
lede: 没有摘要
date: 2023-05-31 23:20:09
thumbnail:
---


首先在配置文件中增加 mysql 配置。
```yaml
mysql:  
  driver: mysql  
  url: localhost  
  port: 3306  
  username: root  
  password: 123456  
  dbname: test
```


```go
package dao  
  
import (  
    "fmt"  
    "github.com/spf13/viper"    
    "gorm.io/driver/mysql"    
    "gorm.io/gorm"
)  
  
type MysqlConfig struct {  
    Url      string `yaml:"url"`  
    Port     int    `yaml:"port"`  
    Username string `yaml:"username"`  
    Password string `yaml:"password"`  
    DbName   string `yaml:"dbname"`  
    Driver   string `yaml:"driver"`  
}  
  
var SqlSession *gorm.DB  
  
func Init() (err error) {  
  
    config := MysqlConfig{  
       viper.GetString("mysql.url"),  
       viper.GetInt("mysql.port"),  
       viper.GetString("mysql.username"),  
       viper.GetString("mysql.password"),  
       viper.GetString("mysql.dbname"),  
       viper.GetString("mysql.driver"),  
    }  
  
    dsn := fmt.Sprintf("%s:%s@(%s:%d)/%s?charset=utf8mb4&parseTime=True&loc=Local",  
       config.Username,  
       config.Password,  
       config.Url,  
       config.Port,  
       config.DbName,  
    )  
  
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})  
    if err != nil {  
       panic(err)  
    }  
    SqlSession = db  
    return  
}  
  
func Close() {  
  
}```


```go
package main

import (  
    "github.com/gin-gonic/gin"
    "github.com/spf13/viper"
)
    
func main() {
	//初始化配置
	if err := settings.Init(); err != nil {  
	    fmt.Printf("config init failed, err:%v \n", err)  
	    return  
	}

    //初始化日志
    if err := logger.Init(); err != nil {  
	    fmt.Printf("logger init failed, err:%v \n", err)  
	    return  
	}

    //初始化数据库
    if err := dao.Init(); err != nil {  
	    fmt.Printf("datasource init failed, err:%v \n", err)  
	    return  
	}

    //初始化路由

    //启动服务器
    r := gin.Default()
    r.GET("/test", func(c *gin.Context) {  
	    c.JSON(200, gin.H{  
	       "message": "success",  
	    })
	})
	err = r.Run(fmt.Sprintf(":%d", viper.GetInt("app.port")))  
	if err != nil {  
		return  
	}
}
```

数据库初始化完成，我们接着创建对象和数据库表。
```go
package entity  
  
import (  
    "gorm.io/gorm"  
    "time"
)  
  
type User struct {  
    Id        int    `json:"id"`  
    Username  string `json:"username"`  
    NickName  string `json:"nick_name"`  
    Avatar    string `json:"avatar"`  
    Password  string `json:"password"`  
    Email     string `json:"email"`  
    Mobile    string `json:"mobile"`  
    DeletedAt gorm.DeletedAt  
    CreatedAt time.Time `gorm:"column:created_at;" json:"created_at"`  
    UpdatedAt time.Time `gorm:"column:updated_at" json:"updated_at"`  
}  
  
func (User) TableName() string {  
    return "sys_user"  
}
```

GROM 默认驼峰来映射表和实体对象，如果想自定义表名可以用 ``TableName()`` 来定义。GORM h还有一些约定字段，使用 CreatedAt，UpdatedAt 会在字段为空时自动填充。DeletedAt 是软删除标识。
对象和数据库准备好之后，我们就可以通过 ``dao.SqlSession`` 来调用。
```go
if err = dao.SqlSession.Debug().Create(u).Error; err != nil {  
    return err  
}

if err := dao.SqlSession.Debug().Offset(pageNum - 1).Limit(pageSize).Find(&roles).Error; err != nil {  
    return nil, err  
}
```

Debug() 会打印 sql 语句，可以帮我们调试，正式环境不建议使用。