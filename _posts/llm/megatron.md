```
wget https://github.com/argoproj/argo-workflows/releases/download/v3.4.11/install.yaml

kubectl create namespace argo
kubectl apply -n argo -f install.yaml
```



[Ebook_Challenges-in-Managing-AI-ML-Workloads-on-Kubernetes.pdf (komodor.com)](https://komodor.com/wp-content/uploads/2023/02/Ebook_Challenges-in-Managing-AI-ML-Workloads-on-Kubernetes.pdf)



[Airflow vs Luigi vs Argo vs Kubeflow vs MLFlow (datarevenue.com)](https://www.datarevenue.com/en-blog/airflow-vs-luigi-vs-argo-vs-mlflow-vs-kubeflow)



[Integrating Argo Workflows and Events – Automated Ramblings (sdbrett.com)](https://sdbrett.com/post/2021-06-18-integrate-argo-wf-events/)



[云原生流水线 Argo Workflows 的安装、使用以及个人体验 - This Cute World](https://thiscute.world/posts/experience-of-argo-workflows/)



[How to Optimize Kubernetes and Argo Workflow Spending | HackerNoon](https://hackernoon.com/how-to-optimize-kubernetes-and-argo-workflow-spending)



大模型（Large Language Model，LLM）。

B，模型参数的个数，1B=10亿。

PT（pre-training）预训练，在较大的数据集上训练一个模型。

SFT（supervised fine-tuning）有监督微调，在预训练的基础上对模型通过有监督学习对模型参数进行调整。

RLHF（Reinforcement Learning from Human Feedback）

LoRA（Low-Rank Adaptation of Large Language Models）冻结预训练好的模型权重参数，在冻结原模型参数的情况下，通过往模型中加入额外的网络层，并只训练这些新增的网络层参数。





DP（Data Parallelism）数据并行，每个GPU上都有完整的模型，将数据分成n份，每个GPU一份数据，各自计算梯度，最后将梯度累加更新模型。DP多用于单机多卡，DDP多用于多机。
数据并行有两个限制： 
	a）超过某一个点之后，每个GPU的batch size变得太小，GPU利用率降低，通信成本增加。 
	b）可使用的最大设备数就是batch size，限制了可用于训练的加速器数量。
对大模型来说因为要在每个GPU上加载完整模型，如果模型过大无法放到单个节点的内存中，那么单纯的数据并行可能是没有意义的，所以数据并行通常会结合模型并行一起使用。

模型并行就是将模型分布在多个GPU上，来解决模型过大个GPU无法放下的问题。模型并行分为两种：
	TP（Tensor parallelism）张量并行，也叫 Intra-Layer Parallelism，是把某一个层做切分，放置到不同设备之上，也可以理解为**把矩阵运算**分配到不同的设备之上，比如把某个矩阵乘法切分成为多个矩阵乘法放到不同设备之上。
	PP（Pipeline Parallelism）流水线并行，也叫 Inter-Layer Parallelism，是把模型**不同的层**放到不同设备之上。

ZeRO

MoE

FSDP



