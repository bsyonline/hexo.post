---
title: Volcano
tags:
  - Distributed Computing
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2018-05-31 23:20:09
thumbna
---

### 介绍

Volcano 是 CNCF 下的一个基于 Kubernetes 的容器批量计算平台，主要用于高性能计算场景。Volcano支持各种调度策略，包括：

- Gang-scheduling

  Gang-scheduling是一种在并发系统中将多个相关联的进程调度到不同处理器上同时运行的策略。其最主要的原则是保证所有相关联的进程能够同时启动，防止部分进程的异常，导致整个关联进程组的阻塞。例如，提交一个批量Job，这个批量Job包含多个任务，要么这多个任务全部调度成功，要么一个都调度不成功。这种All-or-Nothing调度场景，就被称作Gang scheduling。

- Fair-share scheduling

  公平共享调度（Fair-share scheduling）是一种进程调度算法，旨在确保系统中的所有进程都能公平地分享CPU时间。

- Queue scheduling

  队列调度（Queue Scheduling）是一种进程调度算法，它将进程按顺序放入队列中，并按照队列顺序依次执行。这种调度算法通常按照时间片轮转的方式进行，每个进程在执行完一个时间片后被移至队尾，等待下一次执行。队列调度可以避免某些进程长时间等待，确保每个进程都能获得一定的执行时间。

- Preemption scheduling

  Preemption scheduling是一种进程调度算法，它允许操作系统在进程执行过程中进行抢占（preemption），以便将处理器资源分配给其他进程。这种调度算法通常在实时系统中使用，以确保重要任务能够及时获得处理器资源并优先执行。

- Topology-based scheduling

  Topology-based scheduling是一种进程调度算法，它根据进程的拓扑结构进行调度。

- Reclaims

  reclaims指的是将之前分配给对象的内存重新分配给其他对象，以实现内存的重复利用和高效管理。

- Backfill

  backfill通常用于分布式计算系统中的任务调度和负载均衡。例如，当某个节点上的进程崩溃或出现故障时，调度器可以将该节点上未完成的任务重新分配给其他节点进行处理。同样地，当某个节点上的负载过高时，调度器可以将一些任务从该节点上迁移到其他节点上，以平衡整个系统的负载。

- Resource Reservation

  资源预留调度策略（Resource Reservation Scheduling）是一种调度算法，它允许用户或应用程序在将来使用资源之前提前预留这些资源。资源预留调度策略的优点是可以确保所需的资源在需要时可用，从而避免因资源不足而导致的服务质量下降或连接中断等问题。同时，通过提前预留资源，还可以避免在需要时因资源竞争而导致的延迟或等待时间。缺点是降低了资源的整体利用率。此外，资源预留调度策略的实现和维护有一定的成本和复杂性。



### 应用场景

#### 弹性调度

![](D:\Dev\hexo\source\_posts\images\弹性调度.png)

#### 作业拓扑感知调度

![](D:\Dev\hexo\source\_posts\images\拓扑感知调度.png)



#### cpu拓扑感知调度

![](D:\Dev\hexo\source\_posts\images\cpu拓扑感知调度.png)



#### 为spark提供批量调度

![](D:\Dev\hexo\source\_posts\images\为spark提供批量调度.png)



#### 在离线混部

![](D:\Dev\hexo\source\_posts\images\在离线混部.png)



### 架构

<img src="D:\Dev\hexo\source\_posts\images\volcano系统架构.png" style="zoom:50%;" />

<img src="D:\Dev\hexo\source\_posts\images\volcano架构.png" style="zoom:60%;" />

Volcano 由 scheduler、controllermanager、admission 和 vcctl 组成:

- Scheduler Volcano scheduler通过一系列的action和plugin调度Job，并为它找到一个最适合的节点。与Kubernetes default-scheduler相比，Volcano与众不同的 地方是它支持针对Job的多种调度算法。
- Controllermanager Volcano controllermanager管理CRD资源的生命周期。它主要由**Queue ControllerManager**、 **PodGroupControllerManager**、 **VCJob ControllerManager**构成。
- Admission Volcano admission负责对CRD API资源进行校验。
- Vcctl Volcano vcctl是Volcano的命令行客户端工具。

### 安装

#### 安装 volcano

yaml 方式

```
kubectl apply -f https://raw.githubusercontent.com/volcano-sh/volcano/master/installer/volcano-development.yaml
```

查看组件状态

```
# kubectl get all -n volcano-system   
NAME                                       READY   STATUS      RESTARTS   AGE
pod/volcano-admission-7b7dfb976b-hs5t2     1/1     Running     0          118s
pod/volcano-admission-init-7q5xw           0/1     Completed   0          118s
pod/volcano-controllers-7b7bdbbbb7-bd2dt   1/1     Running     0          117s
pod/volcano-scheduler-7cccd7b856-6sk5s     1/1     Running     0          117s

NAME                                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
service/volcano-admission-service   ClusterIP   10.99.164.197   <none>        443/TCP    118s
service/volcano-scheduler-service   ClusterIP   10.98.247.59    <none>        8080/TCP   117s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/volcano-admission     1/1     1            1           118s
deployment.apps/volcano-controllers   1/1     1            1           117s
deployment.apps/volcano-scheduler     1/1     1            1           117s

NAME                                             DESIRED   CURRENT   READY   AGE
replicaset.apps/volcano-admission-7b7dfb976b     1         1         1       118s
replicaset.apps/volcano-controllers-7b7bdbbbb7   1         1         1       117s
replicaset.apps/volcano-scheduler-7cccd7b856     1         1         1       117s

NAME                               COMPLETIONS   DURATION   AGE
job.batch/volcano-admission-init   1/1           24s        118s
```

#### 安装 vcctl

下载源码编译，需要 golang 环境。

```
# make vcctl

# cp _output/bin/vcctl /usr/local/bin

# vcctl -h
Usage:
  vcctl [command]

Available Commands:
  help        Help about any command
  job         vcctl command line operation job
  queue       Queue Operations
  version     Print the version information

Flags:
  -h, --help                           help for vcctl
      --log-flush-frequency duration   Maximum number of seconds between log flushes (default 5s)

Use "vcctl [command] --help" for more information about a command.
```



### volcano 在各场景如何使用

#### AI 场景

使用kubeflow调度pytorch任务

```
# kubectl apply -f kubeflow/training-operator/examples/pytorch/simple.yaml 
pytorchjob.kubeflow.org/pytorch-simple created
# kubectl get pod -n kubeflow
NAME                                READY   STATUS              RESTARTS   AGE
pytorch-simple-master-0             0/1     ContainerCreating   0          15s
pytorch-simple-worker-0             0/1     Init:0/1            0          15s
training-operator-d8799bf5c-kg4jw   1/1     Running             0          42m

# kubectl describe pod pytorch-simple-master-0 -n kubeflow
...
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m39s  default-scheduler  Successfully assigned kubeflow/pytorch-simple-master-0 to k8s-node2
  Normal  Pulling    3m38s  kubelet            Pulling image "docker.io/kubeflowkatib/pytorch-mnist:v1beta1-45c5727"
  Normal  Pulled     3m22s  kubelet            Successfully pulled image "docker.io/kubeflowkatib/pytorch-mnist:v1beta1-45c5727" in 15.845128077s
  Normal  Created    3m22s  kubelet            Created container pytorch
  Normal  Started    3m22s  kubelet            Started container pytorch
```

使用volcano调度pytorch任务

```
apiVersion: "kubeflow.org/v1"
kind: PyTorchJob
metadata:
  name: pytorch-simple
  namespace: kubeflow
spec:
  pytorchReplicaSpecs:
    Master:
      replicas: 1
      restartPolicy: OnFailure
      template:
        spec:
          schedulerName: volcano    #
          containers:
            - name: pytorch
              image: docker.io/kubeflowkatib/pytorch-mnist:v1beta1-45c5727
              imagePullPolicy: Always
              command:
                - "python3"
                - "/opt/pytorch-mnist/mnist.py"
                - "--epochs=1"
    Worker:
      replicas: 1
      restartPolicy: OnFailure
      template:
        spec:
          schedulerName: volcano    #
          containers:
            - name: pytorch
              image: docker.io/kubeflowkatib/pytorch-mnist:v1beta1-45c5727
              imagePullPolicy: Always
              command:
                - "python3"
                - "/opt/pytorch-mnist/mnist.py"
                - "--epochs=1"
```



```
# kubectl describe pod pytorch-simple-master-0 -n kubeflow
...
Events:
  Type    Reason     Age   From     Message
  ----    ------     ----  ----     -------
  Normal  Scheduled  48s   volcano  Successfully assigned kubeflow/pytorch-simple-master-0 to k8s-worker1.sdns.dev.cloud
  Normal  Pulled     48s   kubelet  Container image "docker.io/istio/proxyv2:1.16.0" already present on machine
  Normal  Created    48s   kubelet  Created container istio-init
  Normal  Started    48s   kubelet  Started container istio-init
  Normal  Pulling    47s   kubelet  Pulling image "docker.io/kubeflowkatib/pytorch-mnist:v1beta1-45c5727"
  Normal  Pulled     45s   kubelet  Successfully pulled image "docker.io/kubeflowkatib/pytorch-mnist:v1beta1-45c5727" in 2.141690273s
  Normal  Created    45s   kubelet  Created container pytorch
  Normal  Started    44s   kubelet  Started container pytorch
  Normal  Pulled     44s   kubelet  Container image "docker.io/istio/proxyv2:1.16.0" already present on machine
  Normal  Created    44s   kubelet  Created container istio-proxy
  Normal  Started    44s   kubelet  Started container istio-proxy
```





```
# kubectl describe pod pytorch-dist-mnist-nccl-worker-0
...
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  7m39s                 default-scheduler  Successfully assigned default/pytorch-dist-mnist-nccl-worker-0 to k8s-worker1.sdns.dev.cloud
...
```







#### 大数据场景



#### HPC 场景



### 插件



### 架构设计

#### Queue



#### PodGroup



#### Action



#### Plugin