---
title: window 11 wsl2 安装 ubuntu
date: 2017-05-30 10:22:41
tags:
  - llm
  - AI
category:
  - AI
thumbnail: 
author: bsyonline
lede: 没有摘要
---


开始>启用或关闭windows功能，勾选 Hyper-V、Virtual Machine Platform 和适用于 Linux 的 Windows 子系统，确定，然后重启系统。

如果是家庭版没有 Hyper-V ，保存下面内容为 .bat 文件，cmd 管理员权限运行。

```bat
pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hyper-v.txt
for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hyper-v.txt
Dism /online /enable-feature /featurename:Microsoft-Hyper-V -All /LimitAccess /ALL
pause
```

管理员权限运行 cmd ，执行命令：

```
wls --install
```

安装完成设置用户名密码即可进入子系统。


### 安装 conda

```bash
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O Miniconda3.sh
bash Miniconda3.sh
```

在 ~/.bashrc 最后添加 ``export PATH="/opt/miniconda3/bin:$PATH"`` 然后 `source ~/.bashrc` 。

```bash
conda search python
# 选择需要的版本
conda create -n python310 python=3.10.18
conda activate python310
```


### 安装 LLaMA-Factory

```
cd /mnt/d/Dev/workspace/github/LLaMA-Factory
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
pip install -e ".[torch,metrics]" --no-build-isolation
```

### 配置 debug 环境

安装远程 debug 插件。创建 launch.json 。

```json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "debug_llamafactory",
            "type": "debugpy",
            "request": "launch",
            "module": "llamafactory.cli",
            "args": [
                "train",
                "/mnt/d/Dev/workspace/github/LLaMA-Factory/examples/train_lora/llama3_lora_sft.yaml"
            ]
        }
    ]
}
```

添加 settings.json 配置。

```json
{
	"terminal.integrated.env.linux": {
        "PATH": "/home/rolex/miniconda3/envs/python310/bin:${env:PATH}"
    },
    "python.defaultInterpreterPath": "/home/rolex/miniconda3/envs/python310/bin/python",
    "python.analysis.extraPaths": [
        "/mnt/d/Dev/workspace/github/LLaMA-Factory/src",
        "/home/rolex/miniconda3/envs/python310/lib/python3.10/site-packages"
    ],
    "python.linting.enabled": true,
    "workbench.settings.applyToAllProfiles": [
    ]
}
```

