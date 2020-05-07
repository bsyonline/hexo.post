---
title: Compiling Colidity Contract using Command Line
tags:
  - Blockchain
  - Ethereum
category:
  - Blockchain
author: bsyonline
lede: 没有摘要
date: 2018-03-08 15:38:03
thumbnail:
---

上一篇我们编写了一个简单的智能合约，并使用 remix-browser 工具进行编译。事实上每一个合约代码都需要先经过编译才能运行。Solidity 的编译器有很多种，官方推荐使用 Remix 。remix-browser 就是 Remix 提供的在线工具。使用在线工具的好处是不用安装任何东西，即可快速编写和运行，对于调试简单的合约尤为方便，但是如果合约复杂并且庞大，使用命令行编译更加有效。本篇我们会介绍如何使用命令行编译运行 solidity 。

#### **安装**
如果使用 ubuntu 安装过程会十分简单。详细可参考：[Installing the Solidity Compiler](https://solidity.readthedocs.io/en/develop/installing-solidity.html)

```
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
```

#### **编译合约**

还是编译 helloworld.sol 。

```
cd ~
cd blockchain-samples
mkdir contracts
touch helloworld.sol
```

文件内容如下：

```
pragma solidity ^0.4.0;
contract helloworld {
    string greeting;
    
    function helloworld(string _greeting) public {
        greeting = _greeting;
    }

    function say() constant public returns (string) {
        return greeting;
    }
}
```

执行编译

```
cd ~/`blockchain-samples/contracts
solc -o target --bin --abi helloworld.sol
```

编译完成会生成 .abi 和 .bin 文件。接下来只需要将两个文件中的内容填到模板中。

```
var _greeting = "Hello World" ;
var helloContract = web3.eth.contract(/**.abi文件的内容**/);
var hello = helloContract.new(
   _greeting,
   {
     from: web3.eth.accounts[0], 
     data: '/**0x+.bin文件的内容**/', 
     gas: '燃料值'
   }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
         console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
 })
```

将替换好的内容拷贝到 eth-console 中，就可以完成部署。