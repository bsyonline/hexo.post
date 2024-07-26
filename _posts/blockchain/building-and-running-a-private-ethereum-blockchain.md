---
title: Building and Running a Private Ethereum Blockchain
tags:
  - Blockchain
  - Ethereum
category:
  - Blockchain
author: bsyonline
lede: 没有摘要
date: 2018-03-07 11:21:16
thumbnail:
---

本文介绍如何在 Ubuntu 上安装 private ethereum blockchain ，通过安装私链来初步了解区块链，为我们以后编程做好准备。

#### **安装 ethereum**
首先，需要搭建 ethereum 环境，推荐使用 ubuntu 来完成第一个试验。本文介绍内容均给予 Ubuntu 16.04 LTS 。安装说明参考：[Installation Instructions for Ubuntu](https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Ubuntu)
```
sudo apt-get install software-properties-common
sudo add-apt-repository -y ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install ethereum
```

安装完成后使用 ```geth version``` 来查看 ethereum 的版本，不同的版本配置信息略有差异。

#### **准备工作空间**
```
cd ~
mkdir blockchain-samples
cd blockchain-samples
mkdir node0
cd node0
mkdir config
mkdir data
mkdir scripts
mkdir log
```

#### **定义创世区块**
```
cd ~/blockchain-samples/node0/config
touch genesis-block.json
```
在 genesis-block.json 中写入以下内容：
```
{
   "config": {
        "chainId": 424242,
        "homesteadBlock": 0,
        "eip155Block": 0,
        "eip158Block": 0
    },
    "nonce": "0x0000000000000042",
    "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "difficulty": "0x400",
    "coinbase": "0x3333333333333333333333333333333333333333",
    "timestamp": "0x00",
    "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "gasLimit": "0xffffffff",
    "alloc": {}
}
```

这是私链的初始化配置。

>不同版本的配置略有差异，如果配置信息不正确，在初始化的时候会出现错误。常见错误有：
* Fatal: invalid genesis file: missing 0x prefix for hex data：这个错误信息意思很明白，就是你的 json 文件中，对于16进制数据，需要加上 0x 前缀
* Fatal: invalid genesis file: hex string has odd length: 从 v1.6 开始，设置的十六进制数值，不能是奇数位， 比如不能是 0x0，而应该是 0x00。
* Fatal: failed to write genesis block: genesis has no chain configuration ：这个错误信息，就是说，你的 json 文件中，缺少 config 部分。看到这个信息，我们不需要把 geth 退回到 v1.5 版本，而是需要加上 config 部分。
* Error: invalid sender undefined: 这个错误不会导致初始化失败，但是会在以后的转账（eth.sendTransaction），或者部署智能合约的时候产生。解决方法就是 chainId 不能设置为 0 。 如果你完全按照 github 上给的官方配置文件，就会产生这个错误。


#### **初始化创世区块**
我们可以使用一段 shell 脚本来执行初始化工作。

```
cd ~/blockchain-samples/node0/scripts
touch init_private_chain.sh
chmod +x init_private_chain.sh
```

脚本内容如下：

```
#!/bin/sh

DATADIR=/home/$USER/blockchain-samples/node0/data
GENESIS=/home/$USER/blockchain-samples/node0/config/genesis-block.json
NETWORKID=42
IDENTITY="PrivateChainNode0"
PORT=30303
RPCPORT=8000

# Initialize the private blockchain
geth --networkid $NETWORKID --datadir=$DATADIR --identity $IDENTITY --port $PORT --rpcport $RPCPORT init $GENESIS
```

执行脚本可以看到初始化成功。

#### **启动节点**
初始化成功后，我们使用另外一段脚本来启动节点。
```
cd ~/blockchain-samples/node0/scripts
touch start_node_daemon.sh
chmod +x start_node_daemon.sh
```

脚本内容如下：

```
#!/bin/sh
PORT=30303
RPCPORT=8000
NETWORKID=42
IDENTITY="PrivateChainNode0"
DATADIR=/home/$USER/blockchain-samples/node0/data
NAT=none
RPCADDR="0.0.0.0"

nohup geth --rpc --ws --port $PORT --rpcport $RPCPORT --networkid $NETWORKID --datadir $DATADIR --nat $NAT --identity $IDENTITY --rpcaddr $RPCADDR --wsaddr $RPCADDR --rpccorsdomain * &
```

执行脚本可以启动区块链节点。如果想作为永久节点，可以使用 nohup 来后台运行。

#### **创建账户**

先创建个启动命令行的脚本方便以后使用：

```
cd ~/blockchain-samples/node0/scripts
touch start_console.sh
chmod +x start_console.sh
```

脚本内容如下：

```
#!/bin/sh

PORT=30303
RPCPORT=8545
NETWORKID=42
IDENTITY="PrivateChainNode0"
DATADIR=/home/$USER/blockchain-samples/node0/data
LOGDIR=/home/$USER/blockchain-samples/node0/log
NAT=none
RPCADDR="0.0.0.0"

geth --rpc --rpccorsdomain "*" --rpcapi "db,eth,net,web3" --ws --port $PORT --rpcport $RPCPORT --networkid $NETWORKID --datadir $DATADIR --nat $NAT --identity $IDENTITY --rpcaddr $RPCADDR --wsaddr $RPCADDR console 2>> $LOGDIR/console.log

```


进入 geth 命令行，执行命令查看账户信息：

```
> eth.accounts
[]
> personal.newAccount("rolex")
"0x78cd8b9edb6457cbb8455805e0b8e7edad54cebc"
> eth.accounts
["0x78cd8b9edb6457cbb8455805e0b8e7edad54cebc"]
> eth.accounts
["0x78cd8b9edb6457cbb8455805e0b8e7edad54cebc"]
> personal.newAccount("123456")
"0x9d78bc5b7d1311aa9a4c265a7d7889cc3cd3052a"
> eth.accounts
["0x78cd8b9edb6457cbb8455805e0b8e7edad54cebc", "0x9d78bc5b7d1311aa9a4c265a7d7889cc3cd3052a"]
>
```

#### **挖矿**

```
> miner.start()
null
>
```
查看日志可以看到如下日志：

```
INFO [03-07|16:44:17] Generating DAG in progress               epoch=0 percentage=93 elapsed=2m29.758s
INFO [03-07|16:44:18] Generating DAG in progress               epoch=0 percentage=94 elapsed=2m31.311s
INFO [03-07|16:44:20] Generating DAG in progress               epoch=0 percentage=95 elapsed=2m32.813s
INFO [03-07|16:44:22] Generating DAG in progress               epoch=0 percentage=96 elapsed=2m34.367s
INFO [03-07|16:44:23] Generating DAG in progress               epoch=0 percentage=97 elapsed=2m35.942s
INFO [03-07|16:44:25] Generating DAG in progress               epoch=0 percentage=98 elapsed=2m37.509s
INFO [03-07|16:44:26] Generating DAG in progress               epoch=0 percentage=99 elapsed=2m39.305s
INFO [03-07|16:44:26] Generated ethash verification cache      epoch=0 elapsed=2m39.308s
INFO [03-07|16:44:27] Successfully sealed new block            number=1 hash=b09b14…85ce33
INFO [03-07|16:44:27] 🔨 mined potential block                  number=1 hash=b09b14…85ce33
INFO [03-07|16:44:27] Commit new mining work                   number=2 txs=0 uncles=0 elapsed=174.747µs
```

在挖矿完成前查看账户的余额为 0 ，挖矿完成后，只要账户余额不为 0 ，说明挖矿成功。

```
> acc0=eth.accounts[0]
"0x78cd8b9edb6457cbb8455805e0b8e7edad54cebc"
> eth.getBalance(acc0)
0
> eth.getBalance(acc0)
55000000000000000000
> eth.getBalance(acc0)
1.715e+21
>
```


#### **交易**

我们创建了两个账户，然后来在挖矿获得的代币被存入默认账户（第一个账户）中，我们可以模拟一次交易，将第一个账户的 1 个代币转账到第二个账户中。

```
> amount=web3.toWei(1)
"1000000000000000000"
> eth.sendTransaction({from:acc0, to:acc1, value:amount})
Error: authentication needed: password or unlock
    at web3.js:3143:20
    at web3.js:6347:15
    at web3.js:5081:36
    at <anonymous>:1:1

>
```
转账发生错误，提示账户锁定，所以先解锁账户。
```
> personal.unlockAccount(acc0)
Unlock account 0x78cd8b9edb6457cbb8455805e0b8e7edad54cebc
Passphrase: 
true
> eth.sendTransaction({from:acc0, to:acc1, value:amount})
"0x4af2f15da47d054f167b5f96dfc39c3aa49345111badddca186f24a00204a81f"
> eth.getBalance(acc1)
1000000000000000000
>
```

如果我们没有启动挖矿，那么当交易完成时，我们查看两个账户的余额时会发现没有余额没有变化。这是因为区块链的交易信息需要由矿工挖矿来确认，从而加入到区块链的大账本中。因此，我们只要再次启动 ```miner.start()``` 后在查看账户余额，就可以看到余额的变化了。

