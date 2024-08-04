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

TP（Tensor parallelism）张量并行，

PP（Pipeline Parallelism）流水线并行，

DP（Data Parallelism）数据并行，每个GPU上都有完整的模型，将数据分成n份，每个GPU一份数据，各自计算梯度，最后将梯度累加更新模型。DP多用于单机多卡，DDP多用于多机。

ZeRO

MoE

FSDP



