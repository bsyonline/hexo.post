---
title: go
tags: 
category: []
author: bsyonline
lede: 没有摘要
date: 2023-05-31 23:20:09
thumbnail:
---


目前
```yaml
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  creationTimestamp: null  
  labels:  
    app: gin-study  
  name: gin-study  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      app: kaw  
  strategy: {}  
  template:  
    metadata:  
      creationTimestamp: null  
      labels:  
        app: kaw  
    spec:  
      containers:  
      - image: registry.k8s.com/library/gin-study:v1  
        imagePullPolicy: Always  
        name: gin-study  
        resources: {}  
        volumeMounts:  
        - mountPath: /gin-study/cmd/  
          name: gin-study-config  
      volumes:  
      - name: gin-study-config  
        configMap:  
          name: gin-study-config  
          items:  
          - key: default  
            path: "config.yaml"  
---  
apiVersion: v1  
kind: Service  
metadata:  
  labels:  
    app: gin-study  
  name: gin-study-svc  
spec:  
  ports:  
    - port: 8088  
      protocol: TCP  
      targetPort: 8088  
      nodePort: 31088  
  selector:  
    app: kaw  
  sessionAffinity: None  
  type: NodePort  
  
---  
apiVersion: v1  
kind: ConfigMap  
metadata:  
  name: gin-study-config  
data:  
  default: |  
    app:  
      name: gin-study  
      abc: xyz-use-configmap
```