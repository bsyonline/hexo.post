---
title: PyTorch
tags:
  - Distributed Computing
category:
  - Java
author: bsyonline
lede: 没有摘要
date: 2018-05-31 23:20:09
thumbnail:
---



### PyTorch 安装

[Start Locally | PyTorch](https://pytorch.org/get-started/locally/#windows-anaconda) 选择对应的版本，复制命令安装。

```
conda install pytorch torchvision torchaudio pytorch-cuda=12.1 -c pytorch -c nvidia
```

安装完成，进入 python 环境。

```
import torch # 不报错说明pytorch安装成功
torch.cuda.is_available()  # 检查GPU是否可用，如果输出True，则表示可用
print(torch.__version__)  # 查看torch当前版本
print(torch.version.cuda)  # 编译当前版本的torch使用的cuda版本号
```

