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

SFT（Supervised Fine-Tuning）有监督微调，在预训练的基础上对模型通过有监督学习对模型参数进行调整。

RLHF（Reinforcement Learning from Human Feedback）RLHF 的基本流程包括三个步骤：
	监督微调（SFT）
	奖励模型训练（Reward Model Training）
	强化学习优化（Reinforcement Learning Optimization）​


参数高效微调（Parameter-Efficient Fine-Tuning, PEFT）冻结预训练好的模型权重参数，在冻结原模型参数的情况下，通过往模型中加入额外的网络层，并只训练这些新增的网络层参数。

LoRA（Low-Rank Adaptation of Large Language Models）是 PEFT 的一种，核心思想是通过引入低秩矩阵来更新模型的权重。
LoRA 的实现步骤：
	**选择目标层**：在预训练模型中选择需要微调的层（如 Transformer 的注意力层或全连接层）。
	**引入低秩矩阵**：为每个目标层的权重矩阵引入低秩矩阵 A 和 B。
	**冻结原始权重**：在微调过程中，保持原始权重矩阵 W 不变。
	**更新低秩矩阵**：通过梯度下降优化低秩矩阵 A 和 B。
	**推理阶段**：将低秩矩阵 A⋅B 加到原始权重矩阵 W 上，得到最终的权重矩阵 W′。





DP（Data Parallelism）数据并行，每个GPU上都有完整的模型，将数据分成n份，每个GPU一份数据，各自计算梯度，最后将梯度累加更新模型。DP多用于单机多卡，DDP多用于多机。
数据并行有两个限制： 
	a）超过某一个点之后，每个GPU的batch size变得太小，GPU利用率降低，通信成本增加。 
	b）可使用的最大设备数就是batch size，限制了可用于训练的加速器数量。
对大模型来说因为要在每个GPU上加载完整模型，如果模型过大无法放到单个节点的内存中，那么单纯的数据并行可能是没有意义的，所以数据并行通常会结合模型并行一起使用。

模型并行就是将模型分布在多个GPU上，来解决模型过大个GPU无法放下的问题。模型并行分为两种：
	TP（Tensor parallelism）张量并行，也叫 Intra-Layer Parallelism。
	PP（Pipeline Parallelism）流水线并行，也叫 Inter-Layer Parallelism。

TP（Tensor parallelism）张量并行，是把某一个层做切分，放置到不同设备之上，也可以理解为**把矩阵运算**分配到不同的设备之上，比如把某个矩阵乘法切分成为多个矩阵乘法放到不同设备之上。
模型并行的问题在于：
	a）all-reduce通信单次通信数据量大，并且通信频繁。但是大模型需要分布在多个节点上，节点之间的通信要比机内慢很多。
	b）GPU更适合处理大规模的矩阵运算，但是张量并行把矩阵运算分割到各节点上，运算的规模变小了，数量变多了，故GPU利用率会降低。

TP的原理
对于输入 X 和权重 W：

$$\displaylines{
X\cdot W=Y
}
$$

行并行
将 W 按照行切分为 $W_1$、$W_2$，对应的输入需要切分为 $X_1$、$X_2$，即：

$$\displaylines{
[X_1,X_2]\cdot
\begin{bmatrix}
W_1\\W_2
\end{bmatrix}=X_1 \cdot W_1+X_2 \cdot W_2=Y \\


}

$$

假如有2个 GPU，我们可以将 $X_1 \cdot W_1$ 放到 GPU1 上计算得到结果 $Y_1$，将 $X_2 \cdot W_2$ 放到 GPU2 上计算得到结果 $Y_2$，最后将结果 $Y_1$，$Y_2$ 相加得到最终结果 Y。

列并行
将 W 按照列切分为 $W_1$、$W_2$，输入 X 不需要切分，即

$$\displaylines{ 
X \cdot W_1=Y_1\\ 
X \cdot W_2=Y_2
}$$

将 $X \cdot W_1$ 放到 GPU1 上计算得到结果 $Y_1$，将 $X \cdot W_2$ 放到 GPU2 上计算得到结果 $Y_2$，最后将 $Y_1$ 和
$Y_2$ 按列拼接得到最终结果 Y。

PP（Pipeline Parallelism）流水线并行，也叫 Inter-Layer Parallelism，是把模型**不同的层**放到不同设备之上。

混合并行策略
DP、TP、PP 在模型训练中都有各自的限制，从因此通常需要结合多种并行策略来达到最佳效果。我们记Global Batch Size 为 B ，Microbatch Size 为 b ，n 为 GPU 的数量，每个 PP 包含的 mircrobatch 的数量为 m ，则

$$\displaylines{ 
DP \cdot TP \cdot PP=n \tag{1}
}
$$

$$
m=\frac{1}{b}\cdot\frac{B}{DP} \tag{2}
$$

TP + PP
PP 会产生 bubble ，bubble 的大小为：

$$
bubble\_size=\frac{(PP-1)}{m}\tag{3}
$$

当 DP = 1 时，

$$
PP=\frac{n}{TP}\tag{4}
$$

将（4）带入（2）故

$$
bubble\_size=\frac{(\frac{n}{TP}-1)}{m}\tag{5}
$$

当 TP 增加，bubble size 会减小。但是当 TP 增加时通信也会增加，因为 TP 需要 all-reduce 操作，所以应该尽量减少机间通信。因此我们应该让 TP 小于单台机器的 GPU 的数量，然后使用 PP 来扩展机器规模。

DP + PP

当 TP = 1 时，
$$
PP=\frac{n}{DP}\tag{6}
$$


$$
m=\frac{B}{b}\cdot\frac{1}{DP}\tag{7}
$$


$$
b'=\frac{B}{b}\tag{8}
$$

$$
m=\frac{b'}{DP}\tag{9}
$$

由（3）、（6）、（9）得到

$$
bubble\_size=\frac{(PP-1)}{m}=\frac{(\frac{n}{DP}-1)}{m}=\frac{(\frac{n}{DP}-1)\cdot DP}{b'}=\frac{(n-DP)}{b'}\tag{10}
$$


从（10）看出当 DP 增加，bubble size 变小，但是 bubble 变小 all-reduce 的次数会变多，虽然通信规模不变。

$$
bubble\_size=\frac{(\frac{n}{DP}-1)}{m}\tag{8}
$$



DP + TP






ZeRO（Zero Redundancy Optimizer）是一种用于分布式深度学习训练的优化技术，核心思想是通过消除内存冗余来优化模型训练，从而在大规模模型训练中减少内存占用和通信开销。ZeRO 有三个不同级别，分别对应对 Model States 不同程度的分割 (Paritition)：
	ZeRO-1：分割 `Optimizer States`
	ZeRO-2：分割 `Optimizer States` 与 `Gradients`
	ZeRO-3：分割 `Optimizer States` 、`Gradients` 与 `Parameters`

MoE（Mixture of Experts，专家混合模型）​ 是一种机器学习模型架构，其核心思想是将多个“专家”模型（子模型）组合起来，通过一个“门控网络”（Gating Network）动态地选择最适合的专家来处理不同的输入数据。

FSDP



FlashAttention 是一种高效的注意力机制实现方法，专门针对 Transformer 模型中的自注意力（Self-Attention）计算进行优化。Transformer 模型中的自注意力机制是计算密集型的，尤其是在处理长序列时，其时间和空间复杂度为 O(n2)，其中 n 是序列长度。FlashAttention 旨在减少自注意力计算中的内存占用和计算开销，同时保持模型的性能。FlashAttention 的核心思想是分块计算（Tiling）和重计算（Recomputation）。分块计算是将注意力矩阵的计算分解为多个小块（tiles），逐块计算并累加结果，避免一次性加载整个矩阵。重计算是在前向传播过程中，只保存必要的中间结果，而在反向传播时重新计算需要的中间值。

重计算（Recomputation）是一种在深度学习训练中优化显存使用的技术，核心思想是通过牺牲部分计算时间来减少显存占用。它的基本原理是：在前向传播时只存储必要的输出（如损失值、最终激活值等）不存储所有的中间结果（如每一层的激活值、注意力矩阵等），而是在反向传播时根据需要重新运行前向传播的一部分代码来计算这些中间值。
重计算的实现可以分为两种主要方式：
	**逐层重计算（Layer-wise Recomputation）​**：在前向传播时，只存储每一层的输入和输出，而不存储中间激活值。在反向传播时，根据每一层的输入重新计算该层的激活值。这种方法适用于大多数神经网络结构（如 CNN、RNN 等）。
	**分段重计算（Checkpointing）**：将网络分成若干段（checkpoints），每段包含多个层。在前向传播时，只存储每段的输入和输出，而不存储段内的中间结果。在反向传播时，从最近的 checkpoint 开始重新运行前向传播，生成所需的中间结果。这种方法适用于更复杂的网络结构（如 Transformer）。


Softmax 是一种激活函数，具有归一化、单调性和放大效应的特性。在神经网络中，Softmax 通常作为最后一层的激活函数，将模型的输出转换为类别概率。