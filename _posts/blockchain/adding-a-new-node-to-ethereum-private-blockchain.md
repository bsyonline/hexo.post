---
title: Adding a New Node to Ethereum Private Blockchain
tags:
  - Blockchain
  - Ethereum
category:
  - Blockchain
author: bsyonline
lede: 没有摘要
date: 2018-03-11 11:07:12
thumbnail:
---

前一篇我们介绍了如何搭建 ethereum 私有链，我们当时我们只创建了一个节点，本篇我们将介绍如何将一个新的节点添加到私有链中。

#### **查看节点连接信息**
首先，先启动节点，然后可以查看节点的 enode 信息：

```
> admin.nodeInfo.enode
"enode://6e6052f981fc9748e36df7e3c115a3bc90f49eac91b48bb166bb1f3d59e2d3bb88f6c700295ad30bb7a822c84f881dbfce018f1d677f6f11c1226492a5f38296@[::]:30303?discport=0"
> net.listening
true
> net.peerCount
0
>
```

下一步，我们将在本地在创建一个新的节点。如果要在本地创建多个节点，需要满足以下条件：
* 每个节点要有独立的数据存储路径，即 --datadir 不能相同。
* 每个节点要有独立的端口号，即 --port 和 --rpcport 不能相同。
* ipc 唯一或设置 ipc 不可用。

按照 [Building and Running a Private Ethereum Blockchain](../../../../2018/03/07/building-and-running-a-private-ethereum-blockchain/) 的步骤创建节点，并修改对应的路径和端口号。完成后启动 ethereum-console 查看节点连接信息。

```
> net.listening
true
> net.peerCount
0
> admin.peers
[]
> admin.nodeInfo.enode
"enode://7b84943bcdb8335a60771a7694169d5995831f7429637e6359fe16705c5770634140f177a7fbae3fcd3097843a355c7f0b45c70833a00958d06de291fba11531@[::]:30304?discport=0"
> 
```
>注意这个节点的端口号是 30304 。

#### **添加节点**

保证 2 个节点同时启动，在 ethereum-console 输入命令：

```
> admin.addPeer("enode://7b84943bcdb8335a60771a7694169d5995831f7429637e6359fe16705c5770634140f177a7fbae3fcd3097843a355c7f0b45c70833a00958d06de291fba11531@[::]:30304?discport=0")
true
> net.peerCount

1
> admin.peers

[{
    caps: ["eth/63"],
    id: "7b84943bcdb8335a60771a7694169d5995831f7429637e6359fe16705c5770634140f177a7fbae3fcd3097843a355c7f0b45c70833a00958d06de291fba11531",
    name: "Geth/v1.8.2-stable-b8b9f7f4/linux-amd64/go1.9.4",
    network: {
      inbound: false,
      localAddress: "[::1]:56356",
      remoteAddress: "[::1]:30304",
      static: true,
      trusted: false
    },
    protocols: {
      eth: {
        difficulty: 7988311696,
        head: "0xb971d46360be9329738fe90e54f0baf9e9a58136717694a9a735e9d8eb99558d",
        version: 63
      }
    }
}]
> 
```

可以看到节点已经添加成功。

#### **交易**

最后，我们可以通过转账来测试一下两个节点是否联通。打开两个 ethereum-console node0-console 和 node1-console 。先在 node1-console 中创建账户。

```
> personal.newAccount("123")
"0x207c6bc14b2621bdf660eb651f9d47d90b322289"
> eth.accounts
["0x207c6bc14b2621bdf660eb651f9d47d90b322289"]
>
``` 

再通过 node0-console 给 ```0x207c6bc14b2621bdf660eb651f9d47d90b322289``` 转账 2000 ether 。


```
> eth.sendTransaction({from:'0x3d85328ac7e7306717bdd9d2e6aca00c3ff12941', to:'0x207c6bc14b2621bdf660eb651f9d47d90b322289', value:web3.toWei(1000,'ether')})
"0x357931adaff330c747b40088016b1f41e69203c3d9591eded70645364197332e"
> 
```

等待挖矿确认交易后，即可查看到新节点的账户中多了 2000 ether 。

```
> web3.fromWei(eth.getBalance('0x207c6bc14b2621bdf660eb651f9d47d90b322289'), 'ether')
2000
> 
```