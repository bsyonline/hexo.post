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



### 架构



### volcano-controller 执行流程

cmd/controller-manager/main.go

```
func main() {
	...

	if err := app.Run(s); err != nil {
		fmt.Fprintf(os.Stderr, "%v\n", err)
		os.Exit(1)
	}
}
```

cmd/controller-manager/app/server.go

```
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

这里首先调用了 controller 的 Initialize 方法，然后协程调用 Run() 。framework.Controller 是一个接口，这里实现是 jobcontroller 。

pkg/controllers/job/job_controller.go

```
func (cc *jobcontroller) Run(stopCh <-chan struct{}) {
	...
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

func (cc *jobcontroller) worker(i uint32) {
	klog.Infof("worker %d start ...... ", i)

	for cc.processNextReq(i) {
	}
}

func (cc *jobcontroller) processNextReq(count uint32) bool {
	queue := cc.queueList[count]
	obj, shutdown := queue.Get()
	...

	jobInfo, err := cc.cache.Get(key)
	...
	// 返回的是一个&pendingState
	st := state.NewState(jobInfo)
	...

	// 上边st返回的是一个&pendingState，所以这里的Execute是pendingState中的实现
	if err := st.Execute(action); err != nil {
		...
	}

	// If no error, forget it.
	queue.Forget(req)

	return true
}
```

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

syncJob 主要做了 2 件事情，1 是 initiateJob 的时候创建了 PodGroup ，2 是遍历 JobInfo 的 template 的 TaskInfo ，创建对应的 Pod 。

