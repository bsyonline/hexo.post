---
title: Building a Smart Contract on Ethereum
tags:
  - Blockchain
  - Ethereum
category:
  - Blockchain
author: bsyonline
lede: 没有摘要
date: 2018-03-08 13:54:23
thumbnail:
---

上一篇我们介绍了如何搭建和运行 ethereum 私链，接下来我们会介绍如何基于我们搭建的 ethereum 环境开发智能合约。

#### **智能合约介绍**




#### **编写合约**

官方推荐的编写智能合约的语言是 Solidity ，类似 JavaScript 。官方推荐使用 Remix 来编写简单的合约以及学习 Solidity 。Remix 也提供了在线调试的功能 [remix-browser](https://remix.ethereum.org/#optimize=false&version=soljson-v0.4.21+commit.dfe3193c.js) 。在示例的基础修改，可以开发自己的合约代码。

```
pragma solidity ^0.4.0;
contract hello {
    string greeting;
    
    function hello(string _greeting) public {
        greeting = _greeting;
    }

    function say() constant public returns (string) {
        return greeting;
    }
}
```

#### **部署合约**

写完代码可以直接在浏览器中编译，通过后，将 detail 中的 web3Deploy 部分拷贝并修改名字。

```
var _greeting = "Hello World" ;
var helloContract = web3.eth.contract([{"constant":true,"inputs":[],"name":"say","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"inputs":[{"name":"_greeting","type":"string"}],"payable":false,"stateMutability":"nonpayable","type":"constructor"}]);
var hello = helloContract.new(
   _greeting,
   {
     from: web3.eth.accounts[0], 
     data: '0x6060604052341561000f57600080fd5b6040516102b83803806102b8833981016040528080518201919050508060009080519060200190610041929190610048565b50506100ed565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061008957805160ff19168380011785556100b7565b828001600101855582156100b7579182015b828111156100b657825182559160200191906001019061009b565b5b5090506100c491906100c8565b5090565b6100ea91905b808211156100e65760008160009055506001016100ce565b5090565b90565b6101bc806100fc6000396000f300606060405260043610610041576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063954ab4b214610046575b600080fd5b341561005157600080fd5b6100596100d4565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561009957808201518184015260208101905061007e565b50505050905090810190601f1680156100c65780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6100dc61017c565b60008054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156101725780601f1061014757610100808354040283529160200191610172565b820191906000526020600020905b81548152906001019060200180831161015557829003601f168201915b5050505050905090565b6020604051908101604052806000815250905600a165627a7a723058209c2dcec2c9ed98141a1a6082b04b2bf20f66a53c0daff8ad0596a766f17aa8380029', 
     gas: '4700000'
   }, function (e, contract){
    console.log(e, contract);
    if (typeof contract.address !== 'undefined') {
         console.log('Contract mined! address: ' + contract.address + ' transactionHash: ' + contract.transactionHash);
    }
 })
```

将以上内容拷贝到 geth-console 中，启动挖矿，可以看到合约部署成功。

```
Contract mined! address: 0x451cf68a0950f2affebd7ebaabd212db602a6e62 transactionHash: 0x62025991d86e0eb761c93b304f70b30868d368f5b290bd48f58b601d40094861
```

在 console 中可以执行。

```
> hello
{
  abi: [{
      constant: true,
      inputs: [],
      name: "say",
      outputs: [{...}],
      payable: false,
      stateMutability: "view",
      type: "function"
  }, {
      inputs: [{...}],
      payable: false,
      stateMutability: "nonpayable",
      type: "constructor"
  }],
  address: "0x451cf68a0950f2affebd7ebaabd212db602a6e62",
  transactionHash: "0x62025991d86e0eb761c93b304f70b30868d368f5b290bd48f58b601d40094861",
  allEvents: function(),
  say: function()
}
> hello.say
function()
> hello.say()
"Hello World"
> 
```

