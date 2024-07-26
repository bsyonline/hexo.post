---
title: kub-prometheus-stack配置
date: 2016-08-02 23:29:14
tags:
 - k8s
category: 
 - k8s
thumbnail: 
author: bsyonline
lede: "没有摘要"

---



### 版本关系

|kube-prometheus stack | Kubernetes 1.22 | Kubernetes 1.23 | Kubernetes 1.24 | Kubernetes 1.25 | Kubernetes 1.26 | Kubernetes 1.27 | Kubernetes 1.28|
|---|---|---|---|---|---|---|---|
|release-0.10|	✔|	✔|	✗|	✗|	x|	x|	x|
|release-0.11|	✗|	✔|	✔|	✗|	x|	x|	x|
|release-0.12|	✗|	✗|	✔|	✔|	x|	x|	x|
|release-0.13|	✗|	✗|	✗|	x|	✔|	✔|	✔|
|main| ✗| ✗| ✗| x|	x|	✔|	✔|

根据k8s版本选择kube-prometheus分支。
```
git clone https://github.com/prometheus-operator/kube-prometheus.git -b release-0.12
```

### 镜像修改
kube-prometheus/manifests/kubeStateMetrics-deployment.yaml line35
```
        image: k8s-registry.com/kube-state-metrics/kube-state-metrics:v2.7.0
        #image: registry.k8s.io/kube-state-metrics/kube-state-metrics:v2.7.0
```

kube-prometheus/manifests/prometheusAdapter-deployment.yaml
```
        image: registry.k8s.io/prometheus-adapter/prometheus-adapter:v0.10.0
```



### 暴露服务端口
后续应通过ingress方式暴露。
kube-prometheus/manifests/grafana-service.yaml 

```
spec:
  ports:
  - name: http
    port: 3000
    targetPort: http
    nodePort: 30300
  type: NodePort #add
```

kube-prometheus/manifests/alertmanager-service.yaml 
```
spec:
  ports:
  - name: web
    port: 9093
    targetPort: web
    nodePort: 30093
  - name: reloader-web
    port: 8080
    targetPort: reloader-web
  type: NodePort #add
```

kube-prometheus/manifests/prometheus-service.yaml 
```
spec:
  ports:
  - name: web
    port: 9090
    targetPort: web
    nodePort: 30090
  - name: reloader-web
    port: 8080
    targetPort: reloader-web
  type: NodePort #add
```

ingress方式
新增 ingress.yaml
```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: prom-ingress
  namespace: monitoring
  annotations:
    kubernetes.io/ingress.class: "nginx"
    prometheus.io/http_probe: "true"
spec:
  rules:
  - host: alertmanager.k8s.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: alertmanager-main
            port:
              number: 9093
  - host: grafana.k8s.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: grafana
            port:
              number: 3000
  - host: prometheus.k8s.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: prometheus-k8s
            port:
              number: 9090
```


### 告警规则
#### 新增自定义规则
方式一：
prometheus-prometheusRule.yaml 

```
spec:
  groups:
  - name: prometheus
    rules:
    - alert: MyDiskAlert #add
      annotations:
        description: disk is full.
        summary: disk is full.
      expr: |
        100 * (node_filesystem_size_bytes{fstype=~"xfs|ext4"} - node_filesystem_avail_bytes) / node_filesystem_size_bytes > 0
      for: 1m
      labels:
        severity: critical
```
方式二：
新增文件 test-prometheus-cpuRules.yaml

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: test-cpu-rules
  namespace: monitoring
spec:
  groups:
  - name: cpu
    rules:
    - alert: TestCpuCore
      annotations:
        summary: test cpu cores
        description: test cpu cores
      expr: |
        count(count(node_cpu_seconds_total{instance=~"k8s-master", mode='system'}) by (cpu)) > 0
      for: 10s
      labels:
        severity: critical
```
#### 调整默认告警规则
关闭 Watchdog，kubePrometheus-prometheusRule.yaml
```
spec:
  groups:
  - name: general.rules
    rules:
    - alert: TargetDown
      annotations:
        description: '{{ printf "%.4g" $value }}% of the {{ $labels.job }}/{{ $labels.service }} targets in {{ $labels.namespace }} namespace are down.'
        runbook_url: https://runbooks.prometheus-operator.dev/runbooks/general/targetdown
        summary: One or more targets are unreachable.
      expr: 100 * (count(up == 0) BY (cluster, job, namespace, service) / count(up) BY (cluster, job, namespace, service)) > 10
      for: 10m
      labels:
        severity: warning
#    - alert: Watchdog
#      annotations:
#        description: |
#          This is an alert meant to ensure that the entire alerting pipeline is functional.
#          This alert is always firing, therefore it should always be firing in Alertmanager
#          and always fire against a receiver. There are integrations with various notification
#          mechanisms that send a notification when this alert is not firing. For example the
#          "DeadMansSnitch" integration in PagerDuty.
#        runbook_url: https://runbooks.prometheus-operator.dev/runbooks/general/watchdog
#        summary: An alert that should always be firing to certify that Alertmanager is working properly.
#      expr: vector(1)
#      labels:
#        severity: none
```



### alertmanager告警webhook配置
#### 协作
alertmanager的json格式协作的webhook消息格式不同，无法直接使用，需要有一个中间层进行消息转换。可以使用PrometheusAlert工具。
alertmanager-secret.yaml

```
stringData:
  alertmanager.yaml: |-
    "global":
      "resolve_timeout": "3m"
    templates:
      - 'templates/alert-template.tmpl'
    "receivers":
    - "name": "web.hook.prometheusalert"
      "webhook_configs":
      - "url": "http://192.168.67.204:30092/prometheusalert?type=dd&tpl=prometheus-xz&ddurl=https://xz.wps.cn/api/v1/webhook/send?key=xxx&at=18888888888"
        "send_resolved": true
    "route":
      "group_by":
      - "instance"
      "group_interval": "30s"
      "group_wait": "30s"
      "receiver": "web.hook.prometheusalert"
      "repeat_interval": "12h"
```
使用PrometheusAlert，创建 PrometheusAlert-Deployment.yaml

```
# apiVersion: v1
# kind: Namespace
# metadata:
#   name: monitoring
---  
apiVersion: v1
data:
  app.conf: |
    #---------------------↓全局配置-----------------------
    appname = PrometheusAlert
    #登录用户名
    login_user=prometheusalert
    #登录密码
    login_password=prometheusalert
    #监听地址
    httpaddr = "0.0.0.0"
    #监听端口
    httpport = 8080
    runmode = dev
    #设置代理 proxy = http://123.123.123.123:8080
    proxy =
    #开启JSON请求
    copyrequestbody = true
    #告警消息标题
    title=PrometheusAlert
    #链接到告警平台地址
    GraylogAlerturl=http://graylog.org
    #钉钉告警 告警logo图标地址
    logourl=https://raw.githubusercontent.com/feiyu563/PrometheusAlert/master/doc/alert-center.png
    #钉钉告警 恢复logo图标地址
    rlogourl=https://raw.githubusercontent.com/feiyu563/PrometheusAlert/master/doc/alert-center.png
    #短信告警级别(等于3就进行短信告警) 告警级别定义 0 信息,1 警告,2 一般严重,3 严重,4 灾难
    messagelevel=3
    #电话告警级别(等于4就进行语音告警) 告警级别定义 0 信息,1 警告,2 一般严重,3 严重,4 灾难
    phonecalllevel=4
    #默认拨打号码(页面测试短信和电话功能需要配置此项)
    defaultphone=xxxxxxxx
    #故障恢复是否启用电话通知0为关闭,1为开启
    phonecallresolved=0
    #是否前台输出file or console
    logtype=file
    #日志文件路径
    logpath=logs/prometheusalertcenter.log
    #转换Prometheus,graylog告警消息的时区为CST时区(如默认已经是CST时区，请勿开启)
    prometheus_cst_time=0
    #数据库驱动，支持sqlite3，mysql,postgres如使用mysql或postgres，请开启db_host,db_port,db_user,db_password,db_name的注释
    db_driver=sqlite3
    #db_host=127.0.0.1
    #db_port=3306
    #db_user=root
    #db_password=root
    #db_name=prometheusalert
    #是否开启告警记录 0为关闭,1为开启
    AlertRecord=0
    #是否开启告警记录定时删除 0为关闭,1为开启
    RecordLive=0
    #告警记录定时删除周期，单位天
    RecordLiveDay=7
    # 是否将告警记录写入es7，0为关闭，1为开启
    alert_to_es=0
    # es地址，是[]string
    # beego.Appconfig.Strings读取配置为[]string，使用";"而不是","
    to_es_url=http://localhost:9200
    # to_es_url=http://es1:9200;http://es2:9200;http://es3:9200
    # es用户和密码
    # to_es_user=username
    # to_es_pwd=password
    
    #---------------------↓webhook-----------------------
    #是否开启钉钉告警通道,可同时开始多个通道0为关闭,1为开启
    #协作和钉钉告警通道的消息定义是相同的，可以直接用
    open-dingding=1
    #默认钉钉机器人地址
    ddurl=https://xz.wps.cn/api/v1/webhook/send?key=xxx
    #ddurl=https://oapi.dingtalk.com/robot/send?access_token=xxxxx
    #是否开启 @所有人(0为关闭,1为开启)
    dd_isatall=1
    
    #是否开启微信告警通道,可同时开始多个通道0为关闭,1为开启
    open-weixin=0
    #默认企业微信机器人地址
    wxurl=https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxxxx
    
    #是否开启飞书告警通道,可同时开始多个通道0为关闭,1为开启
    open-feishu=0
    #默认飞书机器人地址
    fsurl=https://open.feishu.cn/open-apis/bot/hook/xxxxxxxxx
    
    #---------------------↓腾讯云接口-----------------------
    #是否开启腾讯云短信告警通道,可同时开始多个通道0为关闭,1为开启
    open-txdx=0
    #腾讯云短信接口key
    TXY_DX_appkey=xxxxx
    #腾讯云短信模版ID 腾讯云短信模版配置可参考 prometheus告警:{1}
    TXY_DX_tpl_id=xxxxx
    #腾讯云短信sdk app id
    TXY_DX_sdkappid=xxxxx
    #腾讯云短信签名 根据自己审核通过的签名来填写
    TXY_DX_sign=腾讯云
    
    #是否开启腾讯云电话告警通道,可同时开始多个通道0为关闭,1为开启
    open-txdh=0
    #腾讯云电话接口key
    TXY_DH_phonecallappkey=xxxxx
    #腾讯云电话模版ID
    TXY_DH_phonecalltpl_id=xxxxx
    #腾讯云电话sdk app id
    TXY_DH_phonecallsdkappid=xxxxx
    
    #---------------------↓华为云接口-----------------------
    #是否开启华为云短信告警通道,可同时开始多个通道0为关闭,1为开启
    open-hwdx=0
    #华为云短信接口key
    HWY_DX_APP_Key=xxxxxxxxxxxxxxxxxxxxxx
    #华为云短信接口Secret
    HWY_DX_APP_Secret=xxxxxxxxxxxxxxxxxxxxxx
    #华为云APP接入地址(端口接口地址)
    HWY_DX_APP_Url=https://rtcsms.cn-north-1.myhuaweicloud.com:10743
    #华为云短信模板ID
    HWY_DX_Templateid=xxxxxxxxxxxxxxxxxxxxxx
    #华为云签名名称，必须是已审核通过的，与模板类型一致的签名名称,按照自己的实际签名填写
    HWY_DX_Signature=华为云
    #华为云签名通道号
    HWY_DX_Sender=xxxxxxxxxx
    
    #---------------------↓阿里云接口-----------------------
    #是否开启阿里云短信告警通道,可同时开始多个通道0为关闭,1为开启
    open-alydx=0
    #阿里云短信主账号AccessKey的ID
    ALY_DX_AccessKeyId=xxxxxxxxxxxxxxxxxxxxxx
    #阿里云短信接口密钥
    ALY_DX_AccessSecret=xxxxxxxxxxxxxxxxxxxxxx
    #阿里云短信签名名称
    ALY_DX_SignName=阿里云
    #阿里云短信模板ID
    ALY_DX_Template=xxxxxxxxxxxxxxxxxxxxxx
    
    #是否开启阿里云电话告警通道,可同时开始多个通道0为关闭,1为开启
    open-alydh=0
    #阿里云电话主账号AccessKey的ID
    ALY_DH_AccessKeyId=xxxxxxxxxxxxxxxxxxxxxx
    #阿里云电话接口密钥
    ALY_DH_AccessSecret=xxxxxxxxxxxxxxxxxxxxxx
    #阿里云电话被叫显号，必须是已购买的号码
    ALY_DX_CalledShowNumber=xxxxxxxxx
    #阿里云电话文本转语音（TTS）模板ID
    ALY_DH_TtsCode=xxxxxxxx
    
    #---------------------↓容联云接口-----------------------
    #是否开启容联云电话告警通道,可同时开始多个通道0为关闭,1为开启
    open-rlydh=0
    #容联云基础接口地址
    RLY_URL=https://app.cloopen.com:8883/2013-12-26/Accounts/
    #容联云后台SID
    RLY_ACCOUNT_SID=xxxxxxxxxxx
    #容联云api-token
    RLY_ACCOUNT_TOKEN=xxxxxxxxxx
    #容联云app_id
    RLY_APP_ID=xxxxxxxxxxxxx
    
    #---------------------↓邮件配置-----------------------
    #是否开启邮件
    open-email=0
    #邮件发件服务器地址
    Email_host=smtp.qq.com
    #邮件发件服务器端口
    Email_port=465
    #邮件帐号
    Email_user=xxxxxxx@qq.com
    #邮件密码
    Email_password=xxxxxx
    #邮件标题
    Email_title=运维告警
    #默认发送邮箱
    Default_emails=xxxxx@qq.com,xxxxx@qq.com
    
    #---------------------↓七陌云接口-----------------------
    #是否开启七陌短信告警通道,可同时开始多个通道0为关闭,1为开启
    open-7moordx=0
    #七陌账户ID
    7MOOR_ACCOUNT_ID=Nxxx
    #七陌账户APISecret
    7MOOR_ACCOUNT_APISECRET=xxx
    #七陌账户短信模板编号
    7MOOR_DX_TEMPLATENUM=n
    #注意：七陌短信变量这里只用一个var1，在代码里写死了。
    #-----------
    #是否开启七陌webcall语音通知告警通道,可同时开始多个通道0为关闭,1为开启
    open-7moordh=0
    #请在七陌平台添加虚拟服务号、文本节点
    #七陌账户webcall的虚拟服务号
    7MOOR_WEBCALL_SERVICENO=xxx
    # 文本节点里被替换的变量，我配置的是text。如果被替换的变量不是text，请修改此配置
    7MOOR_WEBCALL_VOICE_VAR=text
    
    #---------------------↓telegram接口-----------------------
    #是否开启telegram告警通道,可同时开始多个通道0为关闭,1为开启
    open-tg=0
    #tg机器人token
    TG_TOKEN=xxxxx
    #tg消息模式 个人消息或者频道消息 0为关闭(推送给个人)，1为开启(推送给频道)
    TG_MODE_CHAN=0
    #tg用户ID
    TG_USERID=xxxxx
    #tg频道name或者id, 频道name需要以@开始
    TG_CHANNAME=xxxxx
    #tg api地址, 可以配置为代理地址
    #TG_API_PROXY="https://api.telegram.org/bot%s/%s"
    
    #---------------------↓workwechat接口-----------------------
    #是否开启workwechat告警通道,可同时开始多个通道0为关闭,1为开启
    open-workwechat=0
    # 企业ID
    WorkWechat_CropID=xxxxx
    # 应用ID
    WorkWechat_AgentID=xxxx
    # 应用secret
    WorkWechat_AgentSecret=xxxx
    # 接受用户
    WorkWechat_ToUser="zhangsan|lisi"
    # 接受部门
    WorkWechat_ToParty="ops|dev"
    # 接受标签
    WorkWechat_ToTag=""
    # 消息类型, 暂时只支持markdown
    # WorkWechat_Msgtype = "markdown"
    
    #---------------------↓百度云接口-----------------------
    #是否开启百度云短信告警通道,可同时开始多个通道0为关闭,1为开启
    open-baidudx=0
    #百度云短信接口AK(ACCESS_KEY_ID)
    BDY_DX_AK=xxxxx
    #百度云短信接口SK(SECRET_ACCESS_KEY)
    BDY_DX_SK=xxxxx
    #百度云短信ENDPOINT（ENDPOINT参数需要用指定区域的域名来进行定义，如服务所在区域为北京，则为）
    BDY_DX_ENDPOINT=http://smsv3.bj.baidubce.com
    #百度云短信模版ID,根据自己审核通过的模版来填写(模版支持一个参数code：如prometheus告警:{code})
    BDY_DX_TEMPLATE_ID=xxxxx
    #百度云短信签名ID，根据自己审核通过的签名来填写
    TXY_DX_SIGNATURE_ID=xxxxx
    
    #---------------------↓百度Hi(如流)-----------------------
    #是否开启百度Hi(如流)告警通道,可同时开始多个通道0为关闭,1为开启
    open-ruliu=0
    #默认百度Hi(如流)机器人地址
    BDRL_URL=https://api.im.baidu.com/api/msg/groupmsgsend?access_token=xxxxxxxxxxxxxx
    #百度Hi(如流)群ID
    BDRL_ID=123456
    #---------------------↓bark接口-----------------------
    #是否开启telegram告警通道,可同时开始多个通道0为关闭,1为开启
    open-bark=0
    #bark默认地址, 建议自行部署bark-server
    BARK_URL=https://api.day.app
    #bark key, 多个key使用分割
    BARK_KEYS=xxxxx
    # 复制, 推荐开启
    BARK_COPY=1
    # 历史记录保存,推荐开启
    BARK_ARCHIVE=1
    # 消息分组
    BARK_GROUP=PrometheusAlert
    
    #---------------------↓语音播报-----------------------
    #语音播报需要配合语音播报插件才能使用
    #是否开启语音播报通道,0为关闭,1为开启
    open-voice=1
    VOICE_IP=127.0.0.1
    VOICE_PORT=9999
    
    #---------------------↓飞书机器人应用-----------------------
    #是否开启feishuapp告警通道,可同时开始多个通道0为关闭,1为开启
    open-feishuapp=1
    # APPID
    FEISHU_APPID=cli_xxxxxxxxxxxxx
    # APPSECRET
    FEISHU_APPSECRET=xxxxxxxxxxxxxxxxxxxxxx
    # 可填飞书 用户open_id、user_id、union_ids、部门open_department_id
    AT_USER_ID="xxxxxxxx"
  user.csv: |
    2019年4月10日,15888888881,小张,15999999999,备用联系人小陈,15999999998,备用联系人小赵
    2019年4月11日,15888888882,小李,15999999999,备用联系人小陈,15999999998,备用联系人小赵
    2019年4月12日,15888888883,小王,15999999999,备用联系人小陈,15999999998,备用联系人小赵
    2019年4月13日,15888888884,小宋,15999999999,备用联系人小陈,15999999998,备用联系人小赵
kind: ConfigMap
metadata:
  name: prometheus-alert-center-conf
  # namespace: monitoring
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: prometheus-alert-center
    alertname: prometheus-alert-center
  name: prometheus-alert-center
#   namespace: monitoring  
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus-alert-center
      alertname: prometheus-alert-center
  template:
    metadata:
      labels:
        app: prometheus-alert-center
        alertname: prometheus-alert-center
    spec:
      containers:
      - image: feiyu563/prometheus-alert
        name: prometheus-alert-center
        env:
        - name: TZ
          value: "Asia/Shanghai"
        ports:
        - containerPort: 8080
          name: http
        resources:
          limits:
            cpu: 200m
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 100Mi
        volumeMounts:
        - name: prometheus-alert-center-conf-map
          mountPath: /app/conf/app.conf
          subPath: app.conf
        - name: prometheus-alert-center-conf-map
          mountPath: /app/user.csv
          subPath: user.csv
      volumes:
      - name: prometheus-alert-center-conf-map
        configMap:
          name: prometheus-alert-center-conf
          items:
          - key: app.conf
            path: app.conf
          - key: user.csv
            path: user.csv
---
apiVersion: v1
kind: Service
metadata:
  labels:
    alertname: prometheus-alert-center
  name: prometheus-alert-center
#   namespace: monitoring  
  annotations:
    prometheus.io/scrape: 'true'
    prometheus.io/port: '8080'  
spec:
  ports:
  - name: http
    port: 8080
    targetPort: http
  type: NodePort
  selector:
    app: prometheus-alert-center
---
# apiVersion: networking.k8s.io/v1beta1
# kind: Ingress
# metadata:
#   annotations:
#     kubernetes.io/ingress.class: nginx
#   name: prometheus-alert-center
#   namespace: monitoring
# spec:
#   rules:
#     - host: alert-center.local
#       http:
#         paths:
#           - backend:
#               serviceName: prometheus-alert-center
#               servicePort: 8080
#             path: / 

```

#### 服务

alertmanager-secret.yaml

```
stringData:
  alertmanager.yaml: |-
    "global":
      "resolve_timeout": "3m"
    templates:
      - 'templates/alert-template.tmpl'
    "receivers":
    - "name": "webhook"
      "webhook_configs":
      - "url": "http://192.168.67.201:30500/webhook"
        "send_resolved": true
    "route":
      "group_by":
      - "instance"
      "group_interval": "30s"
      "group_wait": "30s"
      "receiver": "webhook"
      "repeat_interval": "12h"
```

### 增加prometheus和grafana的持久化存储
prometheus-prometheus.yaml
```
  serviceMonitorSelector: {}
  version: 2.46.0
  storage:
    volumeClaimTemplate:
      spec:
        storageClassName: dynamic-ceph-rbd
        resources:
          requests:
            storage: 5Gi
```

grafana-deployment.yaml
```
      volumes:
#      - emptyDir: {}
#        name: grafana-storage
      - name: grafana-storage
        persistentVolumeClaim:
          claimName: grafana-data
```

新增 grafana-pvc.yaml
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: grafana-data
  namespace: monitoring
  annotations:
    volume.beta.kubernetes.io/storage-class: "dynamic-ceph-rbd"
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```

### 集成dcgm-exeporter
添加dcgm-exporter.yaml
```
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: "dcgm-exporter"
  namespace: monitoring
  labels:
    app.kubernetes.io/name: "dcgm-exporter"
    app.kubernetes.io/version: "3.2.0"
spec:
  updateStrategy:
    type: RollingUpdate
  selector:
    matchLabels:
      app.kubernetes.io/name: "dcgm-exporter"
      app.kubernetes.io/version: "3.2.0"
  template:
    metadata:
      labels:
        app.kubernetes.io/name: "dcgm-exporter"
        app.kubernetes.io/version: "3.2.0"
      name: "dcgm-exporter"
    spec:
      containers:
      - image: "nvcr.io/nvidia/k8s/dcgm-exporter:3.3.0-3.2.0-ubuntu22.04"
        env:
        - name: "DCGM_EXPORTER_LISTEN"
          value: ":9400"
        - name: "DCGM_EXPORTER_KUBERNETES"
          value: "true"
        name: "dcgm-exporter"
        ports:
        - name: "metrics"
          containerPort: 9400
        securityContext:
          runAsNonRoot: false
          runAsUser: 0
        volumeMounts:
        - name: "pod-gpu-resources"
          readOnly: true
          mountPath: "/var/lib/kubelet/pod-resources"
      volumes:
      - name: "pod-gpu-resources"
        hostPath:
          path: "/var/lib/kubelet/pod-resources"

---

kind: Service
apiVersion: v1
metadata:
  name: "dcgm-exporter"
  namespace: monitoring
  labels:
    app.kubernetes.io/name: "dcgm-exporter"
    app.kubernetes.io/version: "3.2.0"
spec:
  selector:
    app.kubernetes.io/name: "dcgm-exporter"
    app.kubernetes.io/version: "3.2.0"
  ports:
  - name: "metrics"
    port: 9400

```

增加target让Prometheus能够采集到。增加 dcgm-exporter-serviceMonitor.yaml

```
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: "dcgm-exporter"
  namespace: monitoring
  labels:
    app.kubernetes.io/name: "dcgm-exporter"
    app.kubernetes.io/version: "3.2.0"
spec:
  selector:
    matchLabels:
      app.kubernetes.io/name: "dcgm-exporter"
      app.kubernetes.io/version: "3.2.0"
  endpoints:
  - port: "metrics"
    path: "/metrics"
    relabelings:
    - action: replace
      sourceLabels:  [__meta_kubernetes_endpoint_node_name]
      targetLabel: node_name
```



定制指标。修改dcgm-exporter.yaml。

```
        volumeMounts:
        - name: "pod-gpu-resources"
          readOnly: true
          mountPath: "/var/lib/kubelet/pod-resources"
        - name: "gpu-metrics"					#add
          readOnly: true						#add
          mountPath: "/etc/dcgm-exporter"		#add
      volumes:
      - name: "pod-gpu-resources"
        hostPath:
          path: "/var/lib/kubelet/pod-resources"
      - name: "gpu-metrics"						#add
        configMap:								#add
          name: "dcgm-metrics"					#add
```

新增dcgm-metrics.yaml。

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: "dcgm-metrics"
  namespace: monitoring
data:
  # 类属性键；每一个键都映射到一个简单的值
  default-counters.csv: |
    # Format,,
    # If line starts with a '#' it is considered a comment,,
    # DCGM FIELD, Prometheus metric type, help message

    # Clocks,,
    DCGM_FI_DEV_SM_CLOCK,  gauge, SM clock frequency (in MHz).
    DCGM_FI_DEV_MEM_CLOCK, gauge, Memory clock frequency (in MHz).

    # Temperature,,
    DCGM_FI_DEV_MEMORY_TEMP, gauge, Memory temperature (in C).
    DCGM_FI_DEV_GPU_TEMP,    gauge, GPU temperature (in C).

    # Power,,
    DCGM_FI_DEV_POWER_USAGE,              gauge, Power draw (in W).
    DCGM_FI_DEV_TOTAL_ENERGY_CONSUMPTION, counter, Total energy consumption since boot (in mJ).

    # PCIE,,
    DCGM_FI_DEV_PCIE_TX_THROUGHPUT,  counter, Total number of bytes transmitted through PCIe TX (in KB) via NVML.
    DCGM_FI_DEV_PCIE_RX_THROUGHPUT,  counter, Total number of bytes received through PCIe RX (in KB) via NVML.
    DCGM_FI_DEV_PCIE_REPLAY_COUNTER, counter, Total number of PCIe retries.

    # Utilization (the sample period varies depending on the product),,
    DCGM_FI_DEV_GPU_UTIL,      gauge, GPU utilization (in %).
    DCGM_FI_DEV_MEM_COPY_UTIL, gauge, Memory utilization (in %).
    DCGM_FI_DEV_ENC_UTIL,      gauge, Encoder utilization (in %).
    DCGM_FI_DEV_DEC_UTIL ,     gauge, Decoder utilization (in %).

    # Errors and violations,,
    DCGM_FI_DEV_XID_ERRORS,            gauge,   Value of the last XID error encountered.
    # DCGM_FI_DEV_POWER_VIOLATION,       counter, Throttling duration due to power constraints (in us).
    # DCGM_FI_DEV_THERMAL_VIOLATION,     counter, Throttling duration due to thermal constraints (in us).
    # DCGM_FI_DEV_SYNC_BOOST_VIOLATION,  counter, Throttling duration due to sync-boost constraints (in us).
    # DCGM_FI_DEV_BOARD_LIMIT_VIOLATION, counter, Throttling duration due to board limit constraints (in us).
    # DCGM_FI_DEV_LOW_UTIL_VIOLATION,    counter, Throttling duration due to low utilization (in us).
    # DCGM_FI_DEV_RELIABILITY_VIOLATION, counter, Throttling duration due to reliability constraints (in us).

    # Memory usage,,
    DCGM_FI_DEV_FB_FREE, gauge, Framebuffer memory free (in MiB).
    DCGM_FI_DEV_FB_USED, gauge, Framebuffer memory used (in MiB).

    # ECC,,
    # DCGM_FI_DEV_ECC_SBE_VOL_TOTAL, counter, Total number of single-bit volatile ECC errors.
    # DCGM_FI_DEV_ECC_DBE_VOL_TOTAL, counter, Total number of double-bit volatile ECC errors.
    # DCGM_FI_DEV_ECC_SBE_AGG_TOTAL, counter, Total number of single-bit persistent ECC errors.
    # DCGM_FI_DEV_ECC_DBE_AGG_TOTAL, counter, Total number of double-bit persistent ECC errors.

    # Retired pages,,
    # DCGM_FI_DEV_RETIRED_SBE,     counter, Total number of retired pages due to single-bit errors.
    # DCGM_FI_DEV_RETIRED_DBE,     counter, Total number of retired pages due to double-bit errors.
    # DCGM_FI_DEV_RETIRED_PENDING, counter, Total number of pages pending retirement.

    # NVLink,,
    # DCGM_FI_DEV_NVLINK_CRC_FLIT_ERROR_COUNT_TOTAL, counter, Total number of NVLink flow-control CRC errors.
    # DCGM_FI_DEV_NVLINK_CRC_DATA_ERROR_COUNT_TOTAL, counter, Total number of NVLink data CRC errors.
    # DCGM_FI_DEV_NVLINK_REPLAY_ERROR_COUNT_TOTAL,   counter, Total number of NVLink retries.
    # DCGM_FI_DEV_NVLINK_RECOVERY_ERROR_COUNT_TOTAL, counter, Total number of NVLink recovery errors.
    DCGM_FI_DEV_NVLINK_BANDWIDTH_TOTAL,            counter, Total number of NVLink bandwidth counters for all lanes

    # VGPU License status,,
    DCGM_FI_DEV_VGPU_LICENSE_STATUS, gauge, vGPU License status

    # Remapped rows,,
    DCGM_FI_DEV_UNCORRECTABLE_REMAPPED_ROWS, counter, Number of remapped rows for uncorrectable errors
    DCGM_FI_DEV_CORRECTABLE_REMAPPED_ROWS,   counter, Number of remapped rows for correctable errors
    DCGM_FI_DEV_ROW_REMAP_FAILURE,           gauge,   Whether remapping of rows has failed

    #新增4个指标
    DCGM_FI_DEV_COUNT, counter, Number of Devices on the node.
    DCGM_FI_DEV_BRAND,gauge, Device Brand
    DCGM_FI_DEV_PCI_BUSID, gauge, PCI attributes for the device.
    DCGM_FI_DEV_PCIE_LINK_WIDTH, gauge, PCIe Current Link Width.
```



### alertReceiver服务

alertReceiver是跑在k8s上的一个中间服务，用来接收Prometheus告警，触发调用k8s api完成具体操作。


### 部署
```
kubectl apply --server-side -f manifests/setup
kubectl wait \
--for condition=Established \
--all CustomResourceDefinition \
--namespace=monitoring
kubectl apply -f manifests/
```

## helm方式部署
安装kube-prometheus
```
helm repo add gpu-helm-charts https://nvidia.github.io/dcgm-exporter/helm-charts
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

kubectl create namespace monitoring
helm pull bitnami/kube-prometheus
tar -zxf kube-prometheus-8.22.4.tgz

cd kube-prometheus
helm install -f values.yaml kube-prometheus bitnami/kube-prometheus -n monitoring
helm install --generate-name gpu-helm-charts/dcgm-exporter -n monitoring
```
安装kube-prometheus-stack
```
helm pull appstore/kube-prometheus-stack
tar -zxf kube-prometheus-stack-51.2.0.tgz
cd kube-prometheus-stack
helm install -f values.yaml kube-prometheus-stack appstore/kube-prometheus-stack -n monitoring
```
kube-prometheus比kube-prometheus-stack少了grafana

helm方式部署修改配置需要用 jsonnet ，还没有深入研究。