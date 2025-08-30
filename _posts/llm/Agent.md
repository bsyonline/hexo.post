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



CoT（Chain-of-Thought）



Plan-and-Execute



Self-Ask


Thinking and Self-Refection


ReAct





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

