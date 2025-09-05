---
title: Agent
tags:
  - AI
category:
  - AI
author: bsyonline
lede: 没有摘要
date: 2018-05-28 15:32:21
thumbnail:
---


模型处理任务的模式有很多，包括不限于以下：

**CoT（Chain-of-Thought）**

思维链是指将逻辑复杂的问题进行拆解，通过一系列有逻辑关系的思考，形成完整的思考过程。思维链是一种改进的 Prompt 技术，由 google 提出 [Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/pdf/2201.11903)，通过 Prompt 告诉模型开启中间推理过程，比如 "Let’s think step by step" 。

完整的 CoT 的 Prompt 通常包含指令、 逻辑依据和示例。传统的 input 到 output 方式不同，CoT 的过程可以分为 4 个阶段：输入解析 - 思维链生成 - 结果推导 - 输出优化。


**Plan-and-Execute**

Plan-and-Execute 是一种面向复杂任务的大模型任务处理范式。Plan-and-Execute 将复杂任务处理拆解成 “规划 - 执行 - 反馈“ 三个环节，而不是简单的线性推理。


**Self-Ask**

将一个复杂问题拆分为多个简单子问题，通过递归地向自己提问来逐一解答这些子问题，每个子问题都可通过外部工具（主要是搜索引擎）来回答，然后将这些中间答案逐步聚合，最终形成对原始问题的完整解答。

Self-Ask 的 Prompt 中通常包含 “Are follow up questions needed here” 之类的语句。

**Thinking and Self-Refection**

Thinking and Self-Refection 是一种通过模拟人类“主动思考”+“复盘修正”的认知过程，提升大模型推理准确性、逻辑性的任务处理范式。

**ReAct**

ReAct 是 Reasoning + Acting 的缩写，是一种面向大型语言模型智能体的思维与行动协同架构，由 google 提出 [REACT: SYNERGIZING REASONING AND ACTING IN LANGUAGE MODELS](https://arxiv.org/pdf/2210.03629)。ReAct 的核心运行机制包括三步：
- 思考：模型基于当前任务的目标和观察到的信息进行逻辑推理和规划，制定执行策略和动作。
- 行动：根据计划，模型生成执行指令并执行。
- 观察：模型接收并处理执行后的反馈，作为下一轮的输入，帮助模型进行评估和修正后续计划，直到完成任务。




MCP 例子

mcp_server

```python
from fastmcp import FastMCP
import mcp_tools
  
mcp = FastMCP("host-info-mcp")
  
@mcp.tool()
def get_host_info():
	return mcp_tools.get_host_info()
  
def main():
mcp.run("stdio")
  
if __name__ == "__main__":
	main()
```

mcp_tools

```python
import platform
import psutil
import subprocess
import json
  
def get_host_info() -> str:
	"""get host information
	Returns:
	str: the host information in JSON string
	"""
	info: dict[str, str] = {
		"system": platform.system(),
		"release": platform.release(),
		"machine": platform.machine(),
		"processor": platform.processor(),
		"memory_gb": str(round(psutil.virtual_memory().total / (1024**3), 2)),
	}
	  
	cpu_count = psutil.cpu_count(logical=True)
	info["cpu_count"] = str(cpu_count) if cpu_count is not None else "-1"
	
	try:
		cpu_model = subprocess.check_output(
		["sysctl", "-n", "machdep.cpu.brand_string"]
		).decode().strip()
		info["cpu_model"] = cpu_model
	except Exception:
		info["cpu_model"] = "Unknown"
	  
	return json.dumps(info, indent=4)
```


写好 MCP 的server 和 tools 之后，在 trae 的 @Builder with MCP 中注册 tools 。点击【添加更多工具】，如果第一次添加，点【手动添加】，如果不是第一次添加，就能看到之前添加的 MCP 列表，点【添加】，注册新的 MCP 。

```json
{
	"mcpServers": {
		"host-info-mcp": {
		"command": "/opt/anaconda3/envs/python310/bin/python", // 执行的命令
		"args": [
			"mcp_server.py" // 文件路径
		],
		"env": {},
		"cwd": "/mcp" // 程序路径
		}
	}
}
```

按上边格式添加 MCP 配置后，就可以看到添加的 MCP 的状态。添加的工具会默认加到 @Builder with MCP 中，我们直接可以进行测试。

![[mcp-demo.png]]

