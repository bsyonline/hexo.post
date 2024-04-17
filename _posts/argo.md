---
title: Argo
tags:
  - Distributed Computing
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2018-05-31 23:20:09
thumbnail:
---



argo 生态包括 argo workflow，argo cd，argo rollout，argo events 。

### Argo Workflow

#### 简介

Argo Workflow 是一个开源的，容器原生的工作流引擎。它作为 kubernetes CRD, 可以在 k8s 上编排并行作业。

Argo Workflow的一些特点：

- 工作流中的每一步都是一个容器。
- 工作流的依赖通过 DAG 表示。
- 可以方便的在 k8s 上运行科学计算和机器学习任务。
- 不需要复杂的配置就可以基于 CI/CD 流水线在 k8s 上部署应用。

> CRD：Custom Resource Definitions，自定义资源，是一种扩展 k8s 功能的方式。



#### 部署

> 首先需要一个 k8s 集群



```
wget https://github.com/argoproj/argo-workflows/releases/download/v3.4.11/install.yaml

kubectl create namespace argo
kubectl apply -n argo -f install.yaml
```



认证

```
kubectl patch deployment \
  argo-server \
  --namespace argo \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/args", "value": [
  "server",
  "--auth-mode=server"
]}]'
```

UI

```
kubectl -n argo port-forward deployment/argo-server 2746:2746
```

这种方式不通。

```
# kubectl get svc -n argo
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
argo-server   ClusterIP   10.97.133.110   <none>        2746/TCP   11m

# 修改网络通信类型为 NodePort
# kubectl edit svc argo-server -n argo
service/argo-server edited

# kubectl get svc -n argo             
NAME          TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
argo-server   NodePort   10.97.133.110   <none>        2746:31498/TCP   14m
```

然后通过 https://{master ip}:{port} 访问。

CLI

```
# Download the binary
curl -sLO https://github.com/argoproj/argo-workflows/releases/download/v3.4.11/argo-linux-amd64.gz

# Unzip
gunzip argo-linux-amd64.gz

# Make binary executable
chmod +x argo-linux-amd64

# Move binary to path
mv ./argo-linux-amd64 /usr/local/bin/argo

# Test installation
argo version
```



#### Example

官方给出了一个示例程序 hello-world.yaml 。

```
wget https://raw.githubusercontent.com/argoproj/argo-workflows/master/examples/hello-world.yaml
```

看看这个 hello-world 是啥。

```
# vim hello-world.yaml
apiVersion: argoproj.io/v1alpha1
kind: Workflow											# new type of k8s spec
metadata:
  generateName: hello-world-							# name of the workflow spec
  labels:
    workflows.argoproj.io/archive-strategy: "false"
  annotations:
    workflows.argoproj.io/description: |
      This is a simple hello world example.
spec:
  entrypoint: whalesay									# invoke the whalesay template
  templates:
  - name: whalesay										# name of the template
    container:
      image: docker/whalesay:latest
      command: [cowsay]
      args: ["hello world"]
```



提交任务

```
argo submit -n argo --watch hello-world.yaml


kubectl get pods -n argo

argo logs -n argo @latest

```



更复杂一点的示例

```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: say-
spec:
  entrypoint: say
  templates:
  - name: whalesay
    inputs:
      parameters:
      - name: message
    container:
      image: docker/whalesay:latest
      command: ["cowsay", "-f", "{{inputs.parameters.message}}"]
      args: ["hello world"]
  - name: say
    dag:
      tasks:
      - name: A
        template: whalesay
        arguments:
          parameters: [{name: message, value: docker}]
      - name: B
        dependencies: [A]
        template: whalesay
        arguments:
          parameters: [{name: message, value: udder}]
      - name: C
        dependencies: [A]
        template: whalesay
        arguments:
          parameters: [{name: message, value: milk}]
      - name: D
        dependencies: [B, C]
        template: whalesay
        arguments:
          parameters: [{name: message, value: elephant}]
```



```
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  generateName: say-
spec:
  entrypoint: say

  # This spec contains two templates: hello-hello-hello and whalesay
  templates:
    # This is the same template as from the previous example
  - name: whalesay
    inputs:
      parameters:
      - name: message
    container:
      image: docker/whalesay
      command: [cowsay]
      args: ["{{inputs.parameters.message}}"]
  - name: say
    # Instead of just running a container
    # This template has a sequence of steps
    steps:
    - - name: g            # hello1 is run before the following steps
        template: whalesay
        arguments:
          parameters:
          - name: message
            value: "hello1"
    - - name: h           # double dash => run after previous step
        template: whalesay
        arguments:
          parameters:
          - name: message
            value: "hello2a"
      - name: j           # single dash => run in parallel with previous step
        template: whalesay
        arguments:
          parameters:
          - name: message
            value: "hello2b"
    - - name: k            # hello1 is run before the following steps
        template: whalesay
        arguments:
          parameters:
          - name: message
            value: "hello1"
```



```
apiVersion: argoproj.io/v1alpha1
kind: CronWorkflow
metadata:
  name: cron-job
spec:
  # run daily at 11:33 am
  timezone: "Asia/Shanghai"
  schedule: "0/5 * * * ?"
  workflowSpec:
    entrypoint: whalesay
    templates:
      - name: whalesay
        container:
          image: docker/whalesay:latest
          command: [cowsay]
          args: ["hello world"]
```



### Argo CD

#### 简介

Argo CD 是 k8s 上的一个声明式的 GitOps 持续交付工具。

> 什么是CI/CD
>
> 从代码提交开始，依次经历单元测试、代码审查等环节，直到将代码合并到主干分支，这个阶段是集成阶段，这个过程能够自动、快速、随时地进行就是 CI 。将代码部署到对应环境进行测试，并最终发布到生产环境运行，这个阶段就是交付阶段，能够自动、快速、频繁地进行交付就是 CD 。

> 什么是GitOps
>
> GitOps 是 DevOps 的一种最佳实践，即使用 Git 来管理基础设施、管理配置和部署应用。

Argo CD 的特性：

定义、配置、环境都是声明式的，自动化的、支持版本控制。

可以通过多种方式创建：kustomize、helm、jsonnet、yaml、自定义工具等等。

可以根据 git commit 跟踪指定的分支、标签、版本部署到指定环境。

支持 SSO 。

支持回滚到任何 git commit。

支持自动化检查偏差，支持可视化，支持自动/手动同步状态。

支持 webhook 。

### argo rollout



### argo events

```
kubectl create namespace argo-events
kubectl apply -f manifests/install.yaml
kubectl apply -f manifests/install-validating-webhook.yaml
```

eventbus

```
kubectl apply -n argo-events -f examples/eventbus/native.yaml
```

event-source

```
kubectl apply -n argo-events -f examples/event-sources/webhook.yaml
```

sensor rbac

```
kubectl apply -n argo-events -f examples/rbac/sensor-rbac.yaml
```

workflow rbac

```
kubectl apply -n argo-events -f examples/rbac/workflow-rbac.yaml
```

sensor

```
kubectl apply -n argo-events -f examples/sensors/webhook.yaml
```





```
kubectl -n argo-events port-forward $(kubectl -n argo-events get pod -l eventsource-name=webhook -o name) 12000:12000
```



```
curl -d '{"message":"this is my first webhook"}' -H "Content-Type: application/json" -X POST http://localhost:12000/example
```



```
kubectl -n argo-events get workflows | grep "webhook"
```

