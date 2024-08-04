---
title: go
tags: 
category: []
author: bsyonline
lede: 没有摘要
date: 2023-05-31 23:20:09
thumbnail:
---


golang 的日志库很多，这里使 zap 。

首先在配置文件中增加日志配置。
```yaml
logging:  
  level: debug  
  file_name: app.log  
  max_size: 200  
  max_age: 30  
  max_backups: 7  
  compress: true  
  stdout: true
```


```go
package logger  
  
import (  
    "github.com/spf13/viper"  
    "net"    
    "net/http"    
    "net/http/httputil"    
    "os"    
    "path/filepath"    
    "runtime/debug"    
    "strings"    
    "time"  
    "github.com/gin-gonic/gin"    
    "github.com/natefinch/lumberjack"    
    "go.uber.org/zap"    
    "go.uber.org/zap/zapcore"
)
  
var lg *zap.Logger  
  
type LogConfig struct {  
    Level      string `json:"level"`  
    Filename   string `json:"filename"`  
    MaxSize    int    `json:"maxsize"`  
    MaxAge     int    `json:"max_age"`  
    MaxBackups int    `json:"max_backups"`  
    Compress   bool   `json:"compress"`  
    Stdout     bool   `json:"stdout"`  
}  
  
func Init() (err error) {  
    config := LogConfig{  
       Filename:   viper.GetString("logging.file_name"),  
       MaxSize:    viper.GetInt("logging.max_size"),  
       MaxBackups: viper.GetInt("logging.max_backups"),  
       MaxAge:     viper.GetInt("logging.max_age"),  
       Compress:   viper.GetBool("logging.compress"),  
       Stdout:     viper.GetBool("logging.stdout"),  
    }  
    syncers := []zapcore.WriteSyncer{  
       zapcore.AddSync(&lumberjack.Logger{  
          Filename: filepath.Join(config.Filename),  
          MaxSize:  config.MaxSize,  
          MaxAge:   config.MaxAge,  
          Compress: config.Compress,  
       }),  
    }  
    if config.Stdout {  
       syncers = append(syncers, zapcore.AddSync(os.Stdout))  
    }  
    var l = new(zapcore.Level)  
    err = l.UnmarshalText([]byte(viper.GetString("logging.level")))  
    if err != nil {  
       return  
    }  
    core := zapcore.NewCore(getEncoder(), zapcore.NewMultiWriteSyncer(syncers...), l)  
    lg = zap.New(core, zap.AddCaller())  
    zap.ReplaceGlobals(lg)  
    return  
}  
  
func getEncoder() zapcore.Encoder {  
    logTmFmtWithMS := "2006-01-02 15:04:05.000"  
    customTimeEncoder := func(t time.Time, enc zapcore.PrimitiveArrayEncoder) {  
       enc.AppendString(t.Format(logTmFmtWithMS))  
    }  
    customLevelEncoder := func(level zapcore.Level, enc zapcore.PrimitiveArrayEncoder) {  
       enc.AppendString(level.CapitalString())  
    }  
    customCallerEncoder := func(caller zapcore.EntryCaller, enc zapcore.PrimitiveArrayEncoder) {  
       enc.AppendString(caller.TrimmedPath())  
    }  
    encoderConfig := zapcore.EncoderConfig{  
       CallerKey:      "caller_line", // 打印文件名和行数  
       LevelKey:       "level_name",  
       MessageKey:     "msg",  
       TimeKey:        "ts",  
       StacktraceKey:  "stacktrace",  
       LineEnding:     zapcore.DefaultLineEnding,  
       EncodeTime:     customTimeEncoder,   // 自定义时间格式  
       EncodeLevel:    customLevelEncoder,  // 小写编码器  
       EncodeCaller:   customCallerEncoder, // 全路径编码器  
       EncodeDuration: zapcore.SecondsDurationEncoder,  
       EncodeName:     zapcore.FullNameEncoder,  
    }  
    return zapcore.NewConsoleEncoder(encoderConfig)  
}  
  
func GinLogger() gin.HandlerFunc {  
    return func(c *gin.Context) {  
       start := time.Now()  
       path := c.Request.URL.Path  
       query := c.Request.URL.RawQuery  
       c.Next()  
       cost := time.Since(start)  
       lg.Info(path,  
          zap.Int("status", c.Writer.Status()),  
          zap.String("method", c.Request.Method),  
          zap.String("path", path),  
          zap.String("query", query),  
          zap.String("ip", c.ClientIP()),  
          zap.String("errors", c.Errors.ByType(gin.ErrorTypePrivate).String()),  
          zap.Duration("cost", cost),  
       )  
    }  
}  
  
func GinRecovery(stack bool) gin.HandlerFunc {  
    return func(c *gin.Context) {  
       defer func() {  
          if err := recover(); err != nil {  
             // Check for a broken connection, as it is not really a  
             // condition that warrants a panic stack trace.             var brokenPipe bool  
             if ne, ok := err.(*net.OpError); ok {  
                if se, ok := ne.Err.(*os.SyscallError); ok {  
                   if strings.Contains(strings.ToLower(se.Error()), "broken pipe") || strings.Contains(strings.ToLower(se.Error()), "connection reset by peer") {  
                      brokenPipe = true  
                   }  
                }  
             }  
  
             httpRequest, _ := httputil.DumpRequest(c.Request, false)  
             if brokenPipe {  
                lg.Error(c.Request.URL.Path,  
                   zap.Any("error", err),  
                   zap.String("request", string(httpRequest)),  
                )  
                // If the connection is dead, we can't write a status to it.  
                c.Error(err.(error)) // nolint: errcheck  
                c.Abort()  
                return  
             }  
  
             if stack {  
                lg.Error("[Recovery from panic]",  
                   zap.Any("error", err),  
                   zap.String("request", string(httpRequest)),  
                   zap.String("stack", string(debug.Stack())),  
                )  
             } else {  
                lg.Error("[Recovery from panic]",  
                   zap.Any("error", err),  
                   zap.String("request", string(httpRequest)),  
                )  
             }  
             c.AbortWithStatus(http.StatusInternalServerError)  
          }  
       }()  
       c.Next()  
    }  
}
```


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

初始化日志后，我们就可以通过 ``zap.L().Info()`` 来打印。