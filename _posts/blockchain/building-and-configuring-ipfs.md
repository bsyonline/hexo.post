---
title: Building and Configuring IPFS
tags:
  - Blockchain
category:
  - Blockchain
author: bsyonline
lede: 没有摘要
date: 2018-03-15 16:20:53
thumbnail:
---

#### **IPFS 简介**

#### **安装**
```
wget https://dist.ipfs.io/go-ipfs/v0.4.14/go-ipfs_v0.4.14_linux-amd64.tar.gz
tar -zxf go-ipfs_v0.4.14_linux-amd64.tar.gz
cd go-ipfs
./install.sh
```
导入 ipfs 命令成功后可以使用命令查看 ipfs 信息。

```
$ ipfs version
ipfs version 0.4.14
```

 安装好 ipfs 后第一次需要先初始化。

``` 
$ ipfs init
initializing IPFS node at /home/rolex/.ipfs
generating 2048-bit RSA keypair...done
peer identity: QmcePzyBWVKhhi1vhh9CXmAiCEKLQdnRo5vsq3pdaDyQnc
to get started, enter:

	ipfs cat /ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv/readme

$ 
 ```
查看 readme 即可看到如下类似信息：

```
$ ipfs cat /ipfs/QmS4ustL54uo8FzR9455qaxZwuMiUhyvMcX9Ba8nUH4uVv/readme
Hello and Welcome to IPFS!

██╗██████╗ ███████╗███████╗
██║██╔══██╗██╔════╝██╔════╝
██║██████╔╝█████╗  ███████╗
██║██╔═══╝ ██╔══╝  ╚════██║
██║██║     ██║     ███████║
╚═╝╚═╝     ╚═╝     ╚══════╝

If you're seeing this, you have successfully installed
IPFS and are now interfacing with the ipfs merkledag!

 -------------------------------------------------------
| Warning:                                              |
|   This is alpha software. Use at your own discretion! |
|   Much is missing or lacking polish. There are bugs.  |
|   Not yet secure. Read the security notes for more.   |
 -------------------------------------------------------

Check out some of the other files in this directory:

  ./about
  ./help
  ./quick-start     <-- usage examples
  ./readme          <-- this file
  ./security-notes
$ 
```

#### **基本使用**
查看 id 信息

```
$ ipfs id
{
	"ID": "QmcePzyBWVKhhi1vhh9CXmAiCEKLQdnRo5vsq3pdaDyQnc",
	"PublicKey": "CAASpgIwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQC1fFtQLEn4l3nr1ey5RLkmIc3Gx6HTamconhoGPD9Fjw8+KVmH3e52SxH9t3Xxan2ZuG7Eh59IkY13gceakV6vvYvM2WZfipHJ5FKM2KCZ2vukytBV3prU3uk/UszX/ZDJP/IUhJg4HLHjRvaPYlbTEwbi9RwfC10ViGHWmGSitkNaKYwqHkMpZjP5mqfi19ycCN9k0cCGwFophXmR3x9DvlPqP0zctm/GY9hqCVSo5H8c1OCLhO5uNhs1EcpMnY3629pJsLMi03Y4lZaky2YQZLfmb2T18BvQPKSq+byuS2oL6wWam4y8LJIDHevl7y4AOFeVq2w44b3ypW/nkJ+JAgMBAAE=",
	"Addresses": null,
	"AgentVersion": "go-ipfs/0.4.14/",
	"ProtocolVersion": "ipfs/0.1.0"
}
$ 
```

添加文件到 ipfs
```
$ echo 'hello' > website/hello
$ echo 'world' > website/world
$ echo 'hello world' > website/helloworld
$ ipfs add -r website/
added QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN website/hello
added QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o website/helloworld
added QmaRGe7bVmVaLmxbrMiVNXqW4pRNNp3xq7hFtyRKA3mtJL website/world
added Qmb6vsJUAxdxRDg8LrD9fueJSmnwrsUXZWf3rJP7EL19fb website
$ 
```
哈希值是根据文件内容计算的，只要文件内容不变，多次添加地址也不变。

查看文件

```
$ ipfs cat QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
hello world
$ 
```

获取文件
```
$ ipfs get Qmb6vsJUAxdxRDg8LrD9fueJSmnwrsUXZWf3rJP7EL19fb -o foo1
Saving file(s) to foo1
 71 B / 71 B [====================================================================================================================================================================================================================] 100.00% 0s
$ 
```

获取对象
```
$ ipfs object get QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN
{"Links":[],"Data":"\u0008\u0002\u0012\u0006hello\n\u0018\u0006"}
$ ipfs object get QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
{"Links":[],"Data":"\u0008\u0002\u0012\u000chello world\n\u0018\u000c"}
$ ipfs object get QmaRGe7bVmVaLmxbrMiVNXqW4pRNNp3xq7hFtyRKA3mtJL
{"Links":[],"Data":"\u0008\u0002\u0012\u0006world\n\u0018\u0006"}
$ ipfs object get Qmb6vsJUAxdxRDg8LrD9fueJSmnwrsUXZWf3rJP7EL19fb
{"Links":[{"Name":"hello","Hash":"QmZULkCELmmk5XNfCgTnCyFgAVxBRBXyDHGGMVoLFLiXEN","Size":14},{"Name":"helloworld","Hash":"QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o","Size":20},{"Name":"world","Hash":"QmaRGe7bVmVaLmxbrMiVNXqW4pRNNp3xq7hFtyRKA3mtJL","Size":14}],"Data":"\u0008\u0001"}
$ 
```
对象是用 JSON 表示的，```Lisks``` 表示目录，```Data``` 表示数据。

启动 ipfs
```
$ ipfs daemon
Initializing daemon...
Successfully raised file descriptor limit to 2048.
Swarm listening on /ip4/10.12.6.66/tcp/4001
Swarm listening on /ip4/127.0.0.1/tcp/4001
Swarm listening on /ip4/172.17.0.1/tcp/4001
Swarm listening on /ip4/172.18.0.1/tcp/4001
Swarm listening on /ip6/::1/tcp/4001
Swarm listening on /p2p-circuit/ipfs/QmcePzyBWVKhhi1vhh9CXmAiCEKLQdnRo5vsq3pdaDyQnc
Swarm announcing /ip4/10.12.6.66/tcp/4001
Swarm announcing /ip4/127.0.0.1/tcp/4001
Swarm announcing /ip4/172.17.0.1/tcp/4001
Swarm announcing /ip4/172.18.0.1/tcp/4001
Swarm announcing /ip6/::1/tcp/4001
API server listening on /ip4/127.0.0.1/tcp/5001
Gateway (readonly) server listening on /ip4/127.0.0.1/tcp/8080
Daemon is ready
```
启动后可以访问管理界面：[localhost:5001/webui](localhost:5001/webui) 。
使用浏览器也可以查看刚刚添加的文件 [http://localhost:8080/ipfs/Qmb6vsJUAxdxRDg8LrD9fueJSmnwrsUXZWf3rJP7EL19fb](http://localhost:8080/ipfs/Qmb6vsJUAxdxRDg8LrD9fueJSmnwrsUXZWf3rJP7EL19fb)  。

重定向

```
$ ipfs refs -r QmYRMUVULBfj7WrdPESnwnyZmtayN6Sdrwh1nKcQ9QgQeZ
QmT78zSuBmuS4z925WZfrqQ1qHaJ56DQaTfyMUF7F8ff5o
$ 
```

地址绑定

IPFS 是通过文件内容的哈希进行寻找的，所以每次修改文件，文件路径都会改变，为了使用固定的地址来访问，可以将文件地址和节点绑定。

```
$ ipfs name publish Qmb6vsJUAxdxRDg8LrD9fueJSmnwrsUXZWf3rJP7EL19fb
Published to QmcePzyBWVKhhi1vhh9CXmAiCEKLQdnRo5vsq3pdaDyQnc: /ipfs/Qmb6vsJUAxdxRDg8LrD9fueJSmnwrsUXZWf3rJP7EL19fb
```

绑定之后就可以使用 [localhost:8080/ipns/QmcePzyBWVKhhi1vhh9CXmAiCEKLQdnRo5vsq3pdaDyQnc](localhost:8080/ipns/QmcePzyBWVKhhi1vhh9CXmAiCEKLQdnRo5vsq3pdaDyQnc) 来访问。地址是节点固定的，每次修改文件再 publish 一次即可。不过 ipns 的速度实在是很慢啊。

