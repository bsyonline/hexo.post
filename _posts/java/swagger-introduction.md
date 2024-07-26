---
title: 'Swagger : API 设计工具介绍'
date: 2016-10-14 22:15:37
tags:
 - RESTful
 - Swagger
category: 
 - Java
thumbnail: 
author: bsyonline
lede: "没有摘要"
---
Swagger 是一款基于 Node.js 的 API 设计工具，官方网站为 [swagger.io](http://swagger.io) 。Swagger Editor 能直观的生成 API 接口的说明，方便接口开发。

<!--more-->

### Swagger Editor
Editor 可以在线使用 [http://editor.swagger.io/#/](http://editor.swagger.io/#/) ，也可以在本地环境运行，安装也很简单。
```
git clone https://github.com/swagger-api/swagger-editor.git
cd swagger-editor
npm install
npm start
```
访问 `http://localhost:8080/` 可看到效果。
![](https://raw.githubusercontent.com/bsyonline/pic/master/20181014/swagger-editor.png)

swagger 使用 yaml 语法规范，简单的可参考官方示例，语法详细见 [https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md) 。

示例：

```ymal
swagger: "2.0"
info:
  version: 1.0.0
  title: data-engine-api 接口
host: 127.0.0.1:8080
schemes:
  - http
basePath: /query
consumes:
  - application/json
produces:
  - application/json
paths:
  /entInfo:
    get:
      summary: 查询企业基本信息。
      description: description
      parameters:
        - name: entId
          in: query
          description: 唯一标识
          required: true
          type: string
        - name: mask
          in: query
          description: 数据集权限，以','分隔
          required: true
          type: string
        - name: entType
          in: query
          description: 企业类型.
          required: false
          type: string
      responses:
        200:
          description: An array of Characters
          schema:
            $ref: "#/definitions/Info"
          headers:
            Access-Control-Allow-Origin:
              description: The number of allowed requests in the current period
              type: string
            Access-Control-Allow-Headers:
              description: The number of remaining requests in the current period
              type: string
        400:
          description: You tried to teleport. That's just not allowed.
          schema:
            $ref: "#/definitions/Error"
        500:
          description: System error.
          schema:
            $ref: "#/definitions/Error"
        default:
          description: Unexpected errors
          schema:
            $ref: "#/definitions/Error"
  /changeDetails:
    get:
      summary: 变更详情
      description: |
        Gets `Person` objects.
        Optional query param of **size** determines
      parameters:
        - name: uid
          type: string
          required: true
          in: query
          description: 用户id
        - name: pripid
          type: string
          required: true
          in: query
          description: 企业唯一标识
        - name: startDate
          in: query
          required: true
          type: string
          description: 开始时间
        - name: endDate
          in: query
          type: string
          required: true
          description: 结束时间
      responses:
        200:
          description: Successful response
          schema:
            $ref: "#/definitions/ChangeDetails"

definitions:
  ChangeDetails:
    type: object
    properties:
      uid:
        type: string
        description: 用户id
      pripid:
        type: string
        description: 企业唯一标识
      changeDetails:
        type: array
        items:
          $ref: "#/definitions/DataModel"
  Info:
    type: object
    properties:
      basic:
        $ref: "#/definitions/Basic"
      priPerson:
        type: array
        items:
          $ref: "#/definitions/PriPerson"
      shareHolder:
        type: array
        items:
          $ref: "#/definitions/ShareHolder"
      priPersonInv:
        type: array
        items:
          $ref: "#/definitions/PriPersonInv"
      entInv:
        type: array
        items:
          $ref: "#/definitions/EntInv"
      priPersonPosition:
        type: array
        items:
          $ref: "#/definitions/PriPersonPosition"
      sharesImpawn:
        type: array
        items:
          $ref: "#/definitions/SharesImpawn"
      sharesFrost:
        type: array
        items:
          $ref: "#/definitions/SharesFrost"
      disSxbzxr:
        type: array
        items:
          $ref: "#/definitions/DisSxbzxr"
      bzxr:
        type: array
        items:
          $ref: "#/definitions/Bzxr"
      filiation:
        type: array
        items:
          $ref: "#/definitions/Filiation"
      morInfo:
        type: array
        items:
          $ref: "#/definitions/MorInfo"
      morGuaInfo:
        type: array
        items:
          $ref: "#/definitions/MorGuaInfo"
      alter:
        type: array
        items:
          $ref: "#/definitions/Alter"
      liquidation:
        type: array
        items:
          $ref: "#/definitions/Liquidation"
      orgCode:
        type: array
        items:
          $ref: "#/definitions/OrgCode"
      xzcf:
        type: array
        items:
          $ref: "#/definitions/Xzcf"
  Basic:
    type: object
    properties:
      name:
        type: string
        description: 法定代表人姓名
      entName:
        type: string
        description: 企业名称
      identity:
        type: string
        description: 个人识别码
      regNo:
        type: string
        description: 注册号
      oriRegNo:
        type: string
        description: 原注册号
      orgCodes:        
        type: string
        description: 组织机构代码
      regCap:          
        type: string
        description: 注册资本(万元)
      recCap:          
        type: string
        description: 实收资本(万元)
      regCapCur:       
        type: string
        description: 注册资本币种
      esDate:          
        type: string
        description: 开业日期
      opFrom:          
        type: string
        description: 经营期限自
      opTo:            
        type: string
        description: 经营期限至
      entType:         
        type: string
        description: 企业(机构)类型
      entStatus:       
        type: string
        description: 经营状态
      changeDate:      
        type: string
        description: 变更日期
      canDate:         
        type: string
        description: 注销日期
      revDate:         
        type: string
        description: 吊销日期
      dom:             
        type: string
        description: 住址
      abuItem:         
        type: string
        description: 许可经营项目
      cbuItem:         
        type: string
        description: 一般经营项目
      opScope:         
        type: string
        description: 经营(业务)范围
      opScoAndForm:    
        type: string
        description: 经营(业务)范围及方式
      regOrgCode:      
        type: string
        description: 注册地址行政区编号
      regOrgProvince:  
        type: string
        description: 所在省份
      ancheYear:       
        type: string
        description: 最后年检年度
      ancheDate:       
        type: string
        description: 最后年检日期
      industryPhyCode:
        type: string
        description: 行业门类代码
      industryCoCode:  
        type: string
        description: 国民经济行业代码
      empNum:          
        type: string
        description: 员工人数
      entNameEng:      
        type: string
        description: 企业英文名称
      tel:
        type: string
        description: 电话
      creditCode:
        type: string
        description: 信用代码
      province:        
        type: string
        description: 省节点编码
      oploc:           
        type: string
        description: 经营场所
      domDistrict:     
        type: string
        description: 住所所在行政区划
      pripid:
        type: string
        description: 唯一key
  PriPerson:
    type: object
    properties:
      name:
        type: string
        description: 人员姓名
      entName:
        type: string
  ShareHolder:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string
  PriPersonInv:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string
  EntInv:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string
  PriPersonPosition:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string
  SharesImpawn:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string      
  SharesFrost:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string
  DisSxbzxr:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string
  Bzxr:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string
  Filiation:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string
  MorInfo:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string        
  MorGuaInfo:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string
  Alter:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string  
  Liquidation:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string   
  OrgCode:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string        
  Xzcf:
    type: object
    properties:
      name:
        type: string
      entName:
        type: string
  DataModel:
    type: object
    properties:
      pripid:
        type: string
      table:
        type: string
      sequenceId:
        type: string
      changeItems:
        type: array
        items:
          $ref: "#/definitions/ChangeItem"
  ChangeItem:
    type: object
    properties:
      column:
        type: string
      newValue:
        type: string
      oldValue:
        type: string
      type:
        type: string
      date:
        type: string
      own:
        type: string
      key:
        type: string
      apprDate:
        type: string
  Error:
    type: object
    properties:
      code:
        type: string
        description: A machine readable application error code. Not to be confused with the HTTP status code in the response.
      result:
        type: object
```
### Swagger UI
本地安装比较简单，按步骤执行命令应该没有问题，前提需要有 Node.js 环境。
```
git clone git@github.com:swagger-api/swagger-ui.git
cd swagger-ui
npm install
npm run build
npm run serve
```
访问 `http://localhost:8080` 可看到 demo 。
如果有 docker 环境，那使用 docker 部署运行更加简单一些。
```
sudo docker build -t swagger-ui .
sudo docker run -d -p 5000:8080 --name swagger-ui wagger-ui
```
![](https://raw.githubusercontent.com/bsyonline/pic/master/swagger-ui-index.png)


将自己写好的接口文件从 Editor 中导出，保存到 dist/ 下任意位置。
![](https://raw.githubusercontent.com/bsyonline/pic/master/swagger-editor-export.png)

在浏览器中输入路径即可看到自己的接口列表。
![](https://raw.githubusercontent.com/bsyonline/pic/master/swagger-ui-customiz.png)

在浏览器中可以直接进行调试，并可直观看到相关信息。
![](https://raw.githubusercontent.com/bsyonline/pic/master/swagger-ui-post.png)
