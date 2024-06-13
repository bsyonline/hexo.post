---
title: Volcano
tags:
  - Distributed Computing
category:
  - k8s
author: bsyonline
lede: 没有摘要
date: 2018-05-31 23:20:09
thumbnail:
---



### 简介



### 介绍

Volcano 是 CNCF 下的一个基于 Kubernetes 的容器批量计算平台，主要用于高性能计算场景。Volcano 的调度器是基于 kube-batch 实现的。Volcano 2018 年开源，目前最新版本 1.9.0 。[volcano-sh/volcano: A Cloud Native Batch System (Project under CNCF) (github.com)](https://github.com/volcano-sh/volcano)


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
>>>>>>> 40526578e733dc67f5d65f4ceab711f1b104651b



### 架构

<img src="D:\Dev\hexo\source\_posts\images\volcano系统架构.png" style="zoom:50%;" />

**volcano-admission 服务**：通过向 kubernetes 服务注册 MutatingWebhookConfigurations 和 ValidatingWebhookConfigurations，修改及校验 vcjob。主要职责包括如下：
	- mutate：设置 schedulerName，queue，taskName 等。
	- validate：检查 jobSpec 参数，tasks，plugins 等合法性检查。
**volcano-controller 服务**：监听 kube-apiserver 上的所有 vcjob，pod，PodGroup 等资源，主要管理整个 vcjob 的生命周期，包括其相关的 pod 和 PodGroup 的创建，修改（如状态和事件），以及销毁。
**volcano-scheduler 服务**：pod 资源调度及绑定。

### 安装

#### 安装 volcano

yaml 方式

```shell
kubectl apply -f https://raw.githubusercontent.com/volcano-sh/volcano/master/installer/volcano-development.yaml
```

查看组件状态

```shell
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

```shell
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


当我们部署 volcano 完成后，可以看到在 kubernetes 机器上，主要创建了三个 service
![](D:\Dev\hexo\source\_posts\images\volcano组件.png)


### volcano 在各场景如何使用

#### AI 场景

使用kubeflow调度pytorch任务

```shell
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

```yaml
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



```shell
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





```shell
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
#### Gang


#### Binpack
binpack是一种优化算法，以最小化资源使用量为目标，将资源合理地分配给每个任务，使所有资源都可以实现最大化的利用价值。Binpack调度算法的目标是尽量把已有的节点填满（即尽量不往空白节点分配）。具体实现上，Binpack调度算法为满足调度条件的节点打分，节点的资源利用率越高得分越高。

> [!Binpack 分数计算公式]
> score=weight * (request + used) / allocatable
> weight为用户设置的权重
> request为当前pod请求的资源量
> used为当前节点已经分配使用的量
> allocatable为当前节点可用总量

![](https://raw.githubusercontent.com/bsyonline/pic/master/img/4bfa6f7a-1db3-4bc2-a078-23ab4b7a017f.png)
如图所示，集群中存在两个节点，分别为Node 1和Node 2，假设Binpack权重为1，在调度Pod时，Binpack策略对两个节点分别打分。

对Node 1的打分信息如下：

```
CPU Score： 1 * （2 + 4）/ 8 = 0.75
Memory Score： 1 * (4 + 8) / 16 = 0.75
GPU Score：1 * （4 + 4）/ 8 = 1
1 * （0.75 + 0.75 + 1）/（1 + 1 + 1）* 100 = 83.3
```

对Node 2的打分信息如下：

```
CPU Score： 1 * （2 + 6）/ 8 = 1
Memory Score： 1 * (4 + 8) / 16 = 0.75
GPU Score：2 * （4 + 4）/ 8 = 1
1 * （1 + 0.75 + 1）/（1 + 1 + 1）* 100 = 91.6
```

Node 2得分大于Node 1，按照Binpack策略，Pod将会优先调度至Node 2。






### 架构设计

#### Queue
queue 是 volcano 中的一个重要的概念，queue 用来承载 PodGroup，通过 Queue 可以对资源进行更精细的管控（比如不同部门可以使用不同的 Queue 进行资源隔离）。默认使用 default 。 

#### PodGroup
PodGroup 是 volcano 自定义的资源类型，是一组 pod 的集合。通过 PodGroup 可以更方便地协调一组 Pod 的处理。 

#### VolcanoJob
是 Volcano 自定义的Job资源类型，简称 vcjob 。vcjob 是对 PodGroup 的封装，扩展了更丰富的功能。

几种资源的关系：
![](https://raw.githubusercontent.com/bsyonline/pic/master/img/volcano%E5%87%A0%E7%A7%8D%E8%B5%84%E6%BA%90%E7%9A%84%E5%85%B3%E7%B3%BB.png)

#### Action
##### enqueue

入队主要就是过滤出需要处理的Job，根据优先级将所有要处理的Queue加入到一个队列中，同时根据优先级将每一个Queue上的Job也加入到这个Queue的一个队列中，然后根据Queue和Job的优先级来对每一个Job进行预判断，判断当前资源是否满足Job的需求。

##### allocate
就是给每一个Task绑定节点，其处理逻辑主要分以下6步，每次选择一个优先级最高的Task，并找到打分最高的节点bind过去，直到所有的Task都处理完。
- 根据优先级选择一个需要去处理的namespace
- 根据优先级选择一个需要去处理的Queue
- 根据优先级选择一个需要去处理的Job
- 根据优先级选择一个需要去处理的Task
- 过滤去除不满足要求的节点
- 给节点进行打分，并将分数最高的节点bind给这个Task

##### backfill
volcano中为了避免饥饿而有条件地为大作业保留了一些资源，回填是对剩下来未调度小Task进行bind的过程，对于每一个未调度的Task。
- 遍历所有节点，过滤掉不满足要求的node
- 尝试将该Task调度到满足要求的节点上

##### preempt
抢占是一种特殊的Action，它主要处理的场景是当一个高优先级的Task进入调度器但是当前环境中的资源已经无法满足这个Task的时候，需要能将已经调度的任务中驱逐一部分优先级低的Task，以便这个高优先级的Task能够正常运行，因此其处理过程包含选择优先级低的Task并驱逐的逻辑。其处理流程为：
- 对于PodGroup状态不为Pending的Job进行过滤
- 对集群中的Job和Task进行优先级队列的初始化
- 遍历每一个需要进行抢占调度的Task：
  - 对所有节点进行打分和排序。
  - 按照分数排序对每个节点上的Task，判断该Task是否可以抢占（也就是这个Task是否可以驱逐用来腾出资源给待调度的Task），指导找到节点并且可以驱逐的Task腾出来的资源满足待调度的Task为止。
  - 可以跨Queue和Queue内部跨Job之间抢占。


#### Plugin




### Volcano-controller 执行流程

入口 cmd/controller-manager/main.go

```go
func main() {
	...

	if err := app.Run(s); err != nil {
		fmt.Fprintf(os.Stderr, "%v\n", err)
		os.Exit(1)
	}
}
```

cmd/controller-manager/app/server.go

```go
func Run(opt *options.ServerOption) error {
	...

	run := startControllers(config, opt)

	...
	return fmt.Errorf("lost lease")
}

func startControllers(config *rest.Config, opt *options.ServerOption) func(ctx context.Context) {
	...

	return func(ctx context.Context) {
		framework.ForeachController(func(c framework.Controller) {
			if err := c.Initialize(controllerOpt); err != nil {
				...
			}

			go c.Run(ctx.Done())
		})

		<-ctx.Done()
	}
}
```

framework.Controller 是一个接口，pkg/controllers/framework.interface.go，所有的 controller 都实现这个接口。

```go
type Controller interface {
	Name() string
	Initialize(opt *ControllerOption) error
	// Run run the controller
	Run(stopCh <-chan struct{})
}
```

在1.8这个版本 controller 有4个实现：

![image-20240522174602419](D:\Dev\hexo\source\_posts\images\image-20240522174602419.png)

在 startControllers() 中首先调用了 controller 的 Initialize 方法，Initialize 根据 option 初始化名字、队列、cache 等以及注册 handler 函数，然后协程调用 Run() ，在 Run() 中通过 worker 执行具体的 handler ，比如 queuecontroller。

```go
func (c *queuecontroller) Run(stopCh <-chan struct{}) {
	...
	go wait.Until(c.worker, 0, stopCh)
    ...
}

func (c *queuecontroller) worker() {
	for c.processNextReq() {
	}
}
```

每个 controller 实现具体细节不同，但是大体结构一致。

#### QueueController

queuecontroller 负责监听 queue 、podGroup 的状态，来更新 queue 的状态。

在 Run() 中起了两个协程。

```go
func (c *queuecontroller) Run(stopCh <-chan struct{}) {
	...

	go wait.Until(c.worker, 0, stopCh)
	go wait.Until(c.commandWorker, 0, stopCh)

	...
}

func (c *queuecontroller) worker() {
	for c.processNextWorkItem() {
	}
}

func (c *queuecontroller) commandWorker() {
	for c.processNextCommand() {
	}
}

func (c *queuecontroller) processNextWorkItem() bool {
	...
	err := c.syncHandler(req)
	...
	return true
}

func (c *queuecontroller) processNextCommand() bool {
	...
	err := c.syncCommandHandler(cmd)
	...
	return true
}
```

syncHandler 和 syncCommandHandler 在 Initialize 中分别绑定了 handleQueue 和 handleCommand 。handleCommand 中将 command 对象转成 request 对象，加入到 queue 。handleQueue 消费队列中的消息，根据 request 对象的类型调用不同的处理函数来更新 queue 的状态。

```go
func (c *queuecontroller) handleQueue(req *apis.Request) error {
	...

	// 根据queue的状态生成处理函数
	queueState := queuestate.NewState(queue)
	if queueState == nil {
		return fmt.Errorf("queue %s state %s is invalid", queue.Name, queue.Status.State)
	}

	...
	// 根据action和status来执行处理函数
	if err := queueState.Execute(req.Action); err != nil {
		return fmt.Errorf("sync queue %s failed for %v, event is %v, action is %s",
			req.QueueName, err, req.Event, req.Action)
	}

	return nil
}
```



#### PgController

PgController 负责为未指定 PodGroup 的 Pod 分配 PodGroup 。

结构和 QueueController 类似，Run() 中通过协程调用 worker 执行。

```go
func (pg *pgcontroller) Run(stopCh <-chan struct{}) {
	...
	go wait.Until(pg.worker, 0, stopCh)
    ...
}

func (pg *pgcontroller) worker() {
	for pg.processNextReq() {
	}
}

func (pg *pgcontroller) processNextReq() bool {
	...
	// 从list缓存中获取Pod信息
	pod, err := pg.podLister.Pods(req.podNamespace).Get(req.podName)
	if err != nil {
		klog.Errorf("Failed to get pod by <%v> from cache: %v", req, err)
		return true
	}
    ...
	// 对于没有创建对应的PodGroup的Pod创建其PodGroup，并将PodGroup的name写到Pod的annotation中
	if err := pg.createNormalPodPGIfNotExist(pod); err != nil {
		klog.Errorf("Failed to handle Pod <%s/%s>: %v", pod.Namespace, pod.Name, err)
		pg.queue.AddRateLimited(req)
		return true
	}
    ...
	return true
}
```

从 Queue 中取出 Pod ，创建 PodGroup ，将 PodGroup 的 name 更新到 Pod 的 annotation ，具体逻辑在 createNormalPodPGIfNotExist 中。

#### JobController

JobController 也是一样的，在 Run() 中通过协程启动 worker 执行。

```go
func (cc *jobcontroller) Run(stopCh <-chan struct{}) {
	...
	go wait.Until(cc.handleCommands, 0, stopCh)
	var i uint32
	for i = 0; i < cc.workers; i++ {
		go func(num uint32) {
			wait.Until(
				func() {
					cc.worker(num)
				},
				time.Second,
				stopCh)
		}(i)
	}
    ...
}

func (cc *jobcontroller) processNextReq(count uint32) bool {
	queue := cc.queueList[count]
	obj, shutdown := queue.Get()
	...

	jobInfo, err := cc.cache.Get(key)
	...
	// 生成对应的执行函数
	st := state.NewState(jobInfo)
	...

	// 执行
	if err := st.Execute(action); err != nil {
		...
	}

	// If no error, forget it.
	queue.Forget(req)

	return true
}
```

handleCommands 将 command 转换 成 request 对象加入到 Qeueu ，由 worker 进行消费。

NewState 根据 vcjob 状态生成对应的执行函数。

```go
func NewState(jobInfo *apis.JobInfo) State {
	job := jobInfo.Job
	switch job.Status.State.Phase {
	case vcbatch.Pending:
		return &pendingState{job: jobInfo}
	case vcbatch.Running:
		return &runningState{job: jobInfo}
	case vcbatch.Restarting:
		return &restartingState{job: jobInfo}
	case vcbatch.Terminated, vcbatch.Completed, vcbatch.Failed:
		return &finishedState{job: jobInfo}
	case vcbatch.Terminating:
		return &terminatingState{job: jobInfo}
	case vcbatch.Aborting:
		return &abortingState{job: jobInfo}
	case vcbatch.Aborted:
		return &abortedState{job: jobInfo}
	case vcbatch.Completing:
		return &completingState{job: jobInfo}
	}

	// It's pending by default.
	return &pendingState{job: jobInfo}
}
```

vcjob 刚提交到 k8s 状态是 pending，st 返回的就是 &pendingState ，会执行 SyncJob 。

pkg/controllers/job/state/pending.go

```
func (ps *pendingState) Execute(action v1alpha1.Action) error {
	switch action {
		...
	default:
		return SyncJob(ps.job, func(status *vcbatch.JobStatus) bool {
			if ps.job.Job.Spec.MinAvailable <= status.Running+status.Succeeded+status.Failed {
				status.State.Phase = vcbatch.Running
				return true
			}
			return false
		})
	}
}
```

pkg/controllers/job/job_controller_actions.go

```go
func (cc *jobcontroller) syncJob(jobInfo *apis.JobInfo, updateStatus state.UpdateStatusFn) error {
	...
	// job的状态（job.Status.State.Phase）为空或者Pending则进行初始化
	if !isInitiated(job) {
		// 初始化job
		if job, err = cc.initiateJob(job); err != nil {
			return err
		}
	} else {
		...
	}
	...
	// 根据Job中的Tasks的template创建对应的Pod
	for _, ts := range job.Spec.Tasks {
		...
		for i := 0; i < int(ts.Replicas); i++ {
			...
			if pod, found := pods[podName]; !found {
				newPod := createJobPod(job, tc, ts.TopologyPolicy, i, jobForwarding)
				podToCreateEachTask = append(podToCreateEachTask, newPod)
				...
			} else {
				...
			}
		}
		...
	}
	podToCreate[ts.Name] = podToCreateEachTask
	...
	for taskName, podToCreateEachTask := range podToCreate {
		...
		go func(taskName string, podToCreateEachTask []*v1.Pod) {
			...
			for _, pod := range podToCreateEachTask {
				// 协程创建Pod
				go func(pod *v1.Pod) {
					...
					newPod, err := cc.kubeClient.CoreV1().Pods(pod.Namespace).Create(context.TODO(), pod, metav1.CreateOptions{})
					...
				}(pod)
			}
		}(taskName, podToCreateEachTask)
	}
	...
}

func (cc *jobcontroller) initiateJob(job *batch.Job) (*batch.Job, error) {
	...
	if err := cc.createOrUpdatePodGroup(newJob); err != nil {
		...
	}
	...
}

func (cc *jobcontroller) createOrUpdatePodGroup(job *batch.Job) error {
	...
	pg, err = cc.pgLister.PodGroups(job.Namespace).Get(pgName)
	if err != nil {
		...
		pg, err = cc.pgLister.PodGroups(job.Namespace).Get(job.Name)
		if err != nil {
			...
			if _, err = cc.vcClient.SchedulingV1beta1().PodGroups(job.Namespace).Create(context.TODO(), pg, metav1.CreateOptions{}); err != nil {
				...
			}
			return nil
		}
	}
	...
}
```

syncJob 主要做了 2 件事情，1 是 initiateJob 的时候创建了 PodGroup ，2 是遍历 JobInfo 的 template 的 TaskInfo ，创建对应的 Pod 。SyncJob 执行完成 vcjob 的状态就由 pending 变成 running 。

### Volcano-scheduler 执行流程

controller 负责创建出 Pod ，接下来 scheduler 负责 Pod 调度。

scheudler 入口在 cmd/scheduler/main.go 。

```go
func main() {
	...
	if err := app.Run(s); err != nil {
		fmt.Fprintf(os.Stderr, "%v\n", err)
		os.Exit(1)
	}
}
```

cmd/scheduler/app/server.go 。

```go
func Run(opt *options.ServerOption) error {
	...
	ctx := signals.SetupSignalContext()
	run := func(ctx context.Context) {
		sched.Run(ctx.Done())
		<-ctx.Done()
	}
    ...
}
```

/pkg/scheduler/scheduler.go 。

```go
func (pc *Scheduler) Run(stopCh <-chan struct{}) {
	...
	go wait.Until(pc.runOnce, pc.schedulePeriod, stopCh)
}

func (pc *Scheduler) runOnce() {
	...
    actions := pc.actions
	// 创建新的session，将cache、配置复制到session中，为每一个调度器注册对应的调度算法
	ssn := framework.OpenSession(pc.cache, plugins, configurations)
	defer framework.CloseSession(ssn)

	// 根据配置文件中配置的action顺序执行，使用的都是cache中的数据，资源更新不会影响本次执行
	for _, action := range actions {
		actionStartTime := time.Now()
		action.Execute(ssn)
		metrics.UpdateActionDuration(action.Name(), metrics.Duration(actionStartTime))
	}
	...
}
```

在 runOnce 中，每次创建一个新的 session ，在 session 处理过程中配置变化不会影响本次处理。在 session 中有2个主要的动作，OpenSession 和遍历注册的 action 执行。

```go
func OpenSession(cache cache.Cache, tiers []conf.Tier, configurations []conf.Configuration) *Session {
	ssn := openSession(cache)
	ssn.Tiers = tiers
	ssn.Configurations = configurations

	for _, tier := range tiers {
		for _, plugin := range tier.Plugins {
			if pb, found := GetPluginBuilder(plugin.Name); !found {
				klog.Errorf("Failed to get plugin %s.", plugin.Name)
			} else {
				plugin := pb(plugin.Arguments)
				ssn.plugins[plugin.Name()] = plugin
				onSessionOpenStart := time.Now()
				plugin.OnSessionOpen(ssn)
				metrics.UpdatePluginDuration(plugin.Name(), metrics.OnSessionOpen, metrics.Duration(onSessionOpenStart))
			}
		}
	}
	return ssn
}
```

OpenSession 的时候分层加载了 scheduler 的 plugin 调用其 OnSessionOpen 函数。scheduler 的 plugin 全都实现了 Plugin 接口。 

```go
type Plugin interface {
	// The unique name of Plugin.
	Name() string

	OnSessionOpen(ssn *Session)
	OnSessionClose(ssn *Session)
}
```

OnSessionOpen 注册了各种 fn 函数，这些函数实现了 plugin 的各种功能。下表是各插件注册的 fn 。

| function\plugin  | binpack | conformance | drf | gang | nodeorder | predicate | priority | proportion |
| :--------------- | :------ | :---------- | :-- | :--- | :-------- | :-------- | :------- | :--------- |
| jobOrderFn       |         |             | *   | *    |           |           | *        |            |
| queueOrderFn     |         |             |     |      |           |           |          | *          |
| taskOrderFn      |         |             |     |      |           |           | *        |            |
| namespaceOrderFn |         |             | *   |      |           |           |          |            |
| predicateFn      |         |             |     |      |           | *         |          |            |
| nodeOrderFn      | *       |             |     |      | *         |           |          |            |
| batchNodeOrderFn |         |             |     |      | *         |           |          |            |
| preemptableFn    |         | *           | *   | *    |           |           | *        |            |
| reclaimableFn    |         | *           |     | *    |           |           |          | *          |
| overusedFn       |         |             |     |      |           |           |          | *          |
| jobReadyFn       |         |             |     | *    |           |           |          |            |
| jobPipelinedFn   |         |             |     | *    |           |           |          |            |
| jobValidFn       |         |             |     | *    |           |           |          |            |
| jobEnqueueableFn |         |             |     |      |           |           |          | *          |

session 创建好之后，遍历 action 调用其 Execute 函数。volcano 中所有注册的 action 都需要实现该接口。

```go
type Action interface {
	// The unique name of Action.
	Name() string

	// Initialize initializes the allocator plugins.
	Initialize()

	// Execute allocates the cluster's resources into each queue.
	Execute(ssn *Session)

	// UnIntialize un-initializes the allocator plugins.
	UnInitialize()
}
```

在 Execute 中会调用 plugin 的 fn 。 