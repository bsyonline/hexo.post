---
title: Restful API 接口设计最佳实践
date: 2017-02-16 10:27:07
tags:
 - RESTful
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

Restful 原则是由 Roy Fielding 提出的，现在已经成为了普遍遵守的规范。随着 Restful API 的使用也越来越多，如何设计一套优秀的 API 是每一个服务首先需要考虑的事情。

<!-- more -->

我也开发过一些 API ，对 Restful 的理解也是渐进的过程。有过一些经验，也走过一些弯路，借这个机会复复盘。

最开始，对于 Restful 并不了解，看了概念也理解的很肤浅，对于资料中的举例倒是很好理解，但是实际要自己来设计 api 借口的时候，总感觉没还是很模糊，一些举例中没有设计的场景，就抓瞎了。

所以，总结几条建议来帮助设计：
1. 一个 URL 表示一个资源，资源是名词，不要使用动词。
2. 应该尽量保持 api 和 http 规范一致。
3. 名词使用复数形式，不要担心出现不符合语法的使用，比如使用 /persons 要比 /people 要好。
4. 推荐在资源的开始使用版本号，如 /v2/users 。
5. 返回使用 json 而不是 xml 。json 是现在普遍使用的数据格式，且比 xml 更精简。
6. 返回 code 和含义应和 http 状态码保持一致。


下面是一些参考示例：

**1. 通过 id 获取**

```
GET /api/v2/users/1
```

**2. 用户列表**

```
GET /api/v2/users
```

**3. 多个条件查询**

```
GET /api/v2/users?firstName=tom&age=20
```

**4. 创建用户**

```
POST /api/v2/users
Content-Type: application/json

{
  "firstName": "Maggy",
  "lastName": "Miller"
}
```
返回
```
201 Created
Location: /api/v2/users/5197cc
Content-Type: application/json

{
  "id": "5197cc",
  "firstName": "Maggy",
  "lastName": "Miller"
}
```

**5. 分页**
```
GET /api/v2/users?offset=10&limit=25
```

**6. 获取总量**
```
GET /api/v2/users?count=true
```
返回
```
200 OK
Total-Count: 135
Content-Type: application/json

[{
  "id": "5197cc",
  "firstName": "Scott",
  "lastName": "Green"
},{
  "id": "5197cb",
  "firstName": "Maggy",
  "lastName": "Miller"
},...]
```

**7. 信封**
```
GET /api/v2/users/?envelope=true
```
返回
```
200 OK
Content-Type: application/json

{
  "status": 404,
  "response": {
    "message": "Not Found"
  }
}
```

**8. 排序**
```
GET /api/v2/users?sort=-name,+age
```

**9. 状态码**
```
Success codes:

    200 OK - Request succeeded. Response included
    201 Created - Resource created. URL to new resource in Location header
    204 No Content - Request succeeded, but no response body

Error codes:

    400 Bad Request - Could not parse request
    401 Unauthorized - No authentication credentials provided or authentication failed
    403 Forbidden - Authenticated user does not have access
    404 Not Found - Resource not found
    415 Unsupported Media Type - POST/PUT/PATCH request occurred without a application/json content type
    422 Unprocessable Entry - A request to modify or create a resource failed due to a validation error
    429 Too Many Requests - Request rejected due to rate limiting
    500, 501, 502, 503, etc - An internal server error occured

```

以上是常用的一些状态码，更多见 [http://www.restapitutorial.com/httpstatuscodes.html](http://www.restapitutorial.com/httpstatuscodes.html) 。



**10. 属性过滤**
```
GET /api/v2/users?fields=id,firstName
```

**11. 速率限制**
使用秒数更直观。
```
Rate-Limit-Limit - Total credit for current period
Rate-Limit-Remaining - Remaining credit for current period
Rate-Limit-Used - Number of credits used for this request
Rate-Limit-Reset - Number of seconds until the the credit count resets
```
**12. 压缩**


>参考：
1. [http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
2. [http://dev.enchant.com/api/v1](http://dev.enchant.com/api/v1)
