---
title: PyTorch
tags:
  - AI
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2018-05-31 23:20:09
thumbnail:
---



PyTorch 是一个开源的机器学习库，广泛用于计算机视觉和自然语言处理等应用。是目前使用最广泛的深度学习训练框架。

### PyTorch 安装

[Start Locally | PyTorch](https://pytorch.org/get-started/locally/#windows-anaconda) 选择对应的版本，复制命令安装。

```shell
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
```

安装完成，进入 python 环境。

```python
import torch # 不报错说明pytorch安装成功
torch.cuda.is_available()  # 检查GPU是否可用，如果输出True，则表示可用
print(torch.__version__)  # 查看torch当前版本
print(torch.version.cuda)  # 编译当前版本的torch使用的cuda版本号
```


### 数据集



### Tensorboard



### torch

#### Tensor
Pytorch 中的 Tensor 是一种用于存储数据的多维数组。它是构建深度学习模型的基本数据结构，可以包含标量、向量、矩阵等。

```python
import torch

# 创建一个0维Tensor（标量）
scalar_tensor = torch.tensor(1)

# 创建一个1维Tensor（向量）
vector_tensor = torch.tensor([1, 2, 3])

# 创建一个2维Tensor（矩阵）
matrix_tensor = torch.tensor([[1, 2], [3, 4]])

# 创建一个3维Tensor（多维数组）
multi_dimensional_tensor = torch.tensor([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])
```

Tensor 不仅支持多种数据类型，还可以在不同 device 之间相互转换，这使得它在进行大规模并行计算时非常高效。

```python
import torch

# 创建不同数据类型的Tensor
int8_tensor = torch.tensor([1, 2, 3], dtype=torch.int8)
int32_tensor = torch.tensor([1, 2, 3], dtype=torch.int32)
float32_tensor = torch.tensor([1.0, 2.0, 3.0], dtype=torch.float32)
float64_tensor = torch.tensor([1.0, 2.0, 3.0], dtype=torch.float64)

# 将Tensor在CPU和GPU之间移动
if torch.cuda.is_available():
	cpu_tensor = torch.tensor([1, 2, 3])
	gpu_tensor = cpu_tensor.to('cuda')
	print("CPU Tensor:", cpu_tensor)
	print("GPU Tensor:", gpu_tensor)
```


Tensor还可以通过numpy、自定义的特殊方法进行构造

```python
from numpy import array

py_list = [1, 2, 3]

numpy_array = array([1, 2, 3])
# 从列表创建Tensor
pytorch_tensor_from_list = torch.tensor(py_list)
# 从NumPy数组创建Tensor
pytorch_tensor_from_numpy = torch.from_numpy(numpy_array)

numpy_array_2 = pytorch_tensor_from_numpy.numpy()

# 创建具有特定属性的Tensor
empty_tensor = torch.empty(2, 3) # 未初始化的张量
zeros_tensor = torch.zeros(2, 3) # 元素全为0的张量
ones_tensor = torch.ones(2, 3) # 元素全为1的张量
random_tensor = torch.rand(2, 3) # 随机值张量，范围[0, 1)
normal_tensor = torch.randn(2, 3) # 正态分布随机值张量，均值为0，标准差为1
```
### torch.nn

torch.nn 包含了很多模块：
- [Containers](https://pytorch.org/docs/stable/nn.html#containers) 
- [Convolution Layers](https://pytorch.org/docs/stable/nn.html#convolution-layers) 
- [Pooling layers](https://pytorch.org/docs/stable/nn.html#pooling-layers)
- [Padding Layers](https://pytorch.org/docs/stable/nn.html#padding-layers)
- [Non-linear Activations (weighted sum, nonlinearity)](https://pytorch.org/docs/stable/nn.html#non-linear-activations-weighted-sum-nonlinearity)
- [Non-linear Activations (other)](https://pytorch.org/docs/stable/nn.html#non-linear-activations-other)
- [Normalization Layers](https://pytorch.org/docs/stable/nn.html#normalization-layers)
- [Recurrent Layers](https://pytorch.org/docs/stable/nn.html#recurrent-layers)
- [Transformer Layers](https://pytorch.org/docs/stable/nn.html#transformer-layers)
- [Linear Layers](https://pytorch.org/docs/stable/nn.html#linear-layers)
- [Dropout Layers](https://pytorch.org/docs/stable/nn.html#dropout-layers)
- [Sparse Layers](https://pytorch.org/docs/stable/nn.html#sparse-layers)
- [Distance Functions](https://pytorch.org/docs/stable/nn.html#distance-functions)
- [Loss Functions](https://pytorch.org/docs/stable/nn.html#loss-functions)
- [Vision Layers](https://pytorch.org/docs/stable/nn.html#vision-layers)
- [Shuffle Layers](https://pytorch.org/docs/stable/nn.html#shuffle-layers)
- [DataParallel Layers (multi-GPU, distributed)](https://pytorch.org/docs/stable/nn.html#module-torch.nn.parallel)
- [Utilities](https://pytorch.org/docs/stable/nn.html#module-torch.nn.utils)
- [Quantized Functions](https://pytorch.org/docs/stable/nn.html#quantized-functions)
- [Lazy Modules Initialization](https://pytorch.org/docs/stable/nn.html#lazy-modules-initialization)
    - [Aliases](https://pytorch.org/docs/stable/nn.html#aliases)




#### nn.Parameter

nn.Parameter 是一个用于定义可训练参数的类，继承自 `torch.Tensor`，因此它本质上也是一个张量（Tensor），一般在 nn.Module 中定义，参数会在反向传播过程中计算梯度，并且可以通过优化器进行自动更新。

```python
import torch
import torch.nn as nn

class CustomModel(nn.Module):

	def __init__(self):
		super(CustomModel, self).__init__()
		self.weight = nn.Parameter(torch.randn(5, 5))
		self.bias = nn.Parameter(torch.randn(5))
	
	def forward(self, x):
		return x @ self.weight + self.bias

model = CustomModel()
print("模型参数:")
for name, param in model.named_parameters():
	print(f"{name}: {param.shape}")
```

nn.Parameter 和 Tensor 的区别：
	普通 Tensor 不会被自动添加到模型的参数列表中，而 `nn.Parameter` 会。使用 `nn.Parameter` 可以确保张量是模型的一部分，并参与优化。

nn.Parameter 和 buffer 的区别：
	反向传播需要被 optimizer 更新的，称之为 parameter ，比如权重。
	反向传播不需要被 optimizer 更新，称之为 buffer，比如阈值。
#### nn.Module

在 PyTorch 中，`torch.nn.Module` 是所有神经网络模块的基类。它是构建神经网络的核心组件，用于定义网络的结构、参数和前向传播逻辑。几乎所有的神经网络层（如线性层、卷积层、循环层等）和模型都是通过继承 `torch.nn.Module` 来实现的。 `torch.nn.Module` 支持嵌套，可以通过组合多个子模块来构建复杂网络。

一个典型的 `torch.nn.Module` 子类包含以下部分：
- `__init__` **方法**：用于初始化模型的参数和子模块。
- `forward` **方法**：定义模型的前向传播逻辑。

```python
import torch  
from torch import nn  
  
class MyNN(nn.Module):  
  
    def __init__(self) -> None:  
        super().__init__()  
        self.weight = nn.Parameter(torch.randn(5, 5))  
        self.bias = nn.Parameter(torch.randn(5))  
  
    def forward(self, x):  
        output = x + 1  
        return output  
  
myNN = MyNN()  
input = torch.tensor(1.0)  
print(myNN(input))  
print(myNN(3))  
  
print("模型参数:")  
for name, param in myNN.named_parameters():  
    print(f"{name}: {param.shape}")
```

模型保存和加载有两种方式：
第一种：保存模型和参数

```python
# save the model and parameters  
torch.save(myNN, '../model/myNN_v1.pth')  
# load the model  
myNN_v1 = torch.load('../model/myNN_v1.pth')
```

第二种：只保存参数

```python
# save the model's parameters  
torch.save(myNN.state_dict(), '../model/myNN_v2.pth')  
# load the model  
myNN_v2 = MyNN()  
myNN_v2.load_state_dict(torch.load('../model/myNN_v2.pth'))
```

### 卷积层 Convolution Layers

卷积层提供了很多函数如 `nn.Conv2d` ，其主要作用是提取输入数据的特征。


### 池化层 Pooling Layers

池化操作用于降低特征图的空间维度，减少计算量。取池化窗口内的最大值即为最大池化。


#### Padding Layers

卷积会导致像素丢失，比如卷积核的大小为3x3，则会丢失2行2列的像素。如果想要图像在卷积之后保持原始大小，就需要在卷积前进行填充。

### 非线性激活

非线性激活函数可以分为多种类型，常见的包括 `Sigmoid`、`ReLU`、`Leaky ReLU`、`Tanh` 等。

非线性激活主要作用:
	为模型引入非线性的特征。通过对卷积层的输出进行非线性变换，引入非线性特征来提高模型的能力。
	缓解梯度消失的问题。在深层网络中，使用线性激活函数容易导致梯度消失的问题，而非线性激活函数可以通过引入非线性变换，缓解梯度消失问题，有助于提高网络的训练效果。



#### 归一化层 Normalization Layers




#### Recurrent Layers




#### 转换层 Transformer Layers




#### 线性层 Linear Layers



#### Dropout Layers



#### Sparse Layers



#### Distance Functions




#### 损失函数 Loss Functions




#### Vision Layers



#### Shuffle Layers




#### DataParallelism Layers



#### Utilities


#### Quantized Functions


#### Lazy Modules Initialization


#### 优化器




torch.nn 和 torch.nn.functional 的区别：

1. 基本形式差异

- torch.nn: 提供完整的模块类（Module），包含状态和参数

- torch.nn.functional: 提供纯函数操作，无状态

1. 参数管理

- torch.nn: 自动管理参数（如权重和偏置）

- torch.nn.functional: 需要手动管理参数

1. 状态保持

- torch.nn: 可以保持状态（如 BatchNorm 的运行统计信息）

- torch.nn.functional: 无状态，每次调用都是独立的

1. 使用场景

- torch.nn: 适合构建完整的网络模型

- torch.nn.functional: 适合简单操作和函数式编程

1. 性能差异

- torch.nn.functional 通常略快（差异很小）

- torch.nn 提供更好的模型组织和管理

使用建议：

1. 构建模型时优先使用 torch.nn.Module

2. 对于简单操作（如激活函数）可以使用 functional

3. 需要保存状态的层（如 BatchNorm）必须使用 nn.Module

4. 在函数式编程场景中使用 functional