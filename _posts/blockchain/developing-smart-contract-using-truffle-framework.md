---
title: Developing Smart Contract using Truffle Framework
tags:
  - Blockchain
  - Ethereum
category:
  - Blockchain
author: bsyonline
lede: 没有摘要
date: 2018-03-08 19:40:18
thumbnail:
---

#### **truffle 介绍**



#### **安装**

```
npm install -g truffle
npm install -g ethereumjs-testrpc
```


#### **创建项目**

```
cd ~
cd blockchain-samples
mkdir truffle-demo
cd truffle-demo
truffle init
```

在 contracts 下创建合约 ```HelloWorld.js``` 

```
pragma solidity ^0.4.17;

contract HelloWorld {
    function say() constant public returns (string) {
        return "hello world";
    }
}
```

在 migrations 下创建 ```2_initial_helloworld.js``` 

```
var helloWorld = artifacts.require("./HelloWorld.sol");

module.exports = function(deployer) {
  deployer.deploy(helloWorld);
};
```


在 ```truffle.js``` 中添加网路配置

```
module.exports = {
  // See <http://truffleframework.com/docs/advanced/configuration>
  // to customize your Truffle configuration!
  networks: {
    dev: {
      host: "localhost", //本地地址，因为是在本机上建立的节点
      port: 8545,        //Ethereum的rpc监听的端口号，默认是8545
      network_id: 999    // 自定义网络号
    }
  }
};
```


#### **编译**

```
truffle compile
truffle migrate --network dev --reset
```

#### **调用合约**

```
truffle comsole --network dev
```

```
truffle(dev)> HelloWorld.deployed().then(instance => helloworld = instance)
TruffleContract {
  constructor: 
   { [Function: TruffleContract]
     _static_methods: 
      { setProvider: [Function: setProvider],
        new: [Function: new],
        at: [Function: at],
        deployed: [Function: deployed],
        defaults: [Function: defaults],
        hasNetwork: [Function: hasNetwork],
        isDeployed: [Function: isDeployed],
        detectNetwork: [Function: detectNetwork],
        setNetwork: [Function: setNetwork],
        resetAddress: [Function: resetAddress],
        link: [Function: link],
        clone: [Function: clone],
        addProp: [Function: addProp],
        toJSON: [Function: toJSON] },
     _properties: 
      { contract_name: [Object],
        contractName: [Object],
        abi: [Object],
        network: [Function: network],
        networks: [Function: networks],
        address: [Object],
        transactionHash: [Object],
        links: [Function: links],
        events: [Function: events],
        binary: [Function: binary],
        deployedBinary: [Function: deployedBinary],
        unlinked_binary: [Object],
        bytecode: [Object],
        deployedBytecode: [Object],
        sourceMap: [Object],
        deployedSourceMap: [Object],
        source: [Object],
        sourcePath: [Object],
        legacyAST: [Object],
        ast: [Object],
        compiler: [Object],
        schema_version: [Function: schema_version],
        schemaVersion: [Function: schemaVersion],
        updated_at: [Function: updated_at],
        updatedAt: [Function: updatedAt] },
     _property_values: {},
     _json: 
      { contractName: 'HelloWorld',
        abi: [Object],
        bytecode: '0x6060604052341561000f57600080fd5b6101578061001e6000396000f300606060405260043610610041576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063954ab4b214610046575b600080fd5b341561005157600080fd5b6100596100d4565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561009957808201518184015260208101905061007e565b50505050905090810190601f1680156100c65780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6100dc610117565b6040805190810160405280600b81526020017f68656c6c6f20776f726c64000000000000000000000000000000000000000000815250905090565b6020604051908101604052806000815250905600a165627a7a72305820fcc0cd665f3f7d4bc38115abb0cab0cd48aa0640db2bdc8e3efcdeea6c35c5fd0029',
        deployedBytecode: '0x606060405260043610610041576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063954ab4b214610046575b600080fd5b341561005157600080fd5b6100596100d4565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561009957808201518184015260208101905061007e565b50505050905090810190601f1680156100c65780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6100dc610117565b6040805190810160405280600b81526020017f68656c6c6f20776f726c64000000000000000000000000000000000000000000815250905090565b6020604051908101604052806000815250905600a165627a7a72305820fcc0cd665f3f7d4bc38115abb0cab0cd48aa0640db2bdc8e3efcdeea6c35c5fd0029',
        sourceMap: '26:113:0:-;;;;;;;;;;;;;;;;;',
        deployedSourceMap: '26:113:0:-;;;;;;;;;;;;;;;;;;;;;;;;52:85;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;23:1:-1;8:100;33:3;30:1;27:2;8:100;;;99:1;94:3;90;84:5;80:1;75:3;71;64:6;52:2;49:1;45:3;40:15;;8:100;;;12:14;3:109;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;52:85:0;92:6;;:::i;:::-;110:20;;;;;;;;;;;;;;;;;;;;52:85;:::o;26:113::-;;;;;;;;;;;;;;;:::o',
        source: 'pragma solidity ^0.4.17;\n\ncontract HelloWorld {\n    function say() constant public returns (string) {\n        return "hello world";\n    }\n}\n',
        sourcePath: '/home/rolex/blockchain-samples/dapps/truffle-demo/contracts/HelloWorld.sol',
        ast: [Object],
        legacyAST: [Object],
        compiler: [Object],
        networks: [Object],
        schemaVersion: '2.0.0',
        updatedAt: '2018-03-08T11:23:39.505Z' },
     setProvider: [Function: bound setProvider],
     new: [Function: bound new],
     at: [Function: bound at],
     deployed: [Function: bound deployed],
     defaults: [Function: bound defaults],
     hasNetwork: [Function: bound hasNetwork],
     isDeployed: [Function: bound isDeployed],
     detectNetwork: [Function: bound detectNetwork],
     setNetwork: [Function: bound setNetwork],
     resetAddress: [Function: bound resetAddress],
     link: [Function: bound link],
     clone: [Function: bound clone],
     addProp: [Function: bound addProp],
     toJSON: [Function: bound toJSON],
     web3: 
      Web3 {
        _requestManager: [Object],
        currentProvider: [Object],
        eth: [Object],
        db: [Object],
        shh: [Object],
        net: [Object],
        personal: [Object],
        bzz: [Object],
        settings: [Object],
        version: [Object],
        providers: [Object],
        _extend: [Object] },
     class_defaults: 
      { from: '0x78cd8b9edb6457cbb8455805e0b8e7edad54cebc',
        gas: 6721975,
        gasPrice: 100000000000 },
     currentProvider: 
      HttpProvider {
        host: 'http://localhost:8545',
        timeout: 0,
        user: undefined,
        password: undefined,
        headers: undefined,
        send: [Function],
        sendAsync: [Function],
        _alreadyWrapped: true },
     network_id: '999' },
  abi: 
   [ { constant: true,
       inputs: [],
       name: 'say',
       outputs: [Object],
       payable: false,
       stateMutability: 'view',
       type: 'function' } ],
  contract: 
   Contract {
     _eth: 
      Eth {
        _requestManager: [Object],
        getBalance: [Object],
        getStorageAt: [Object],
        getCode: [Object],
        getBlock: [Object],
        getUncle: [Object],
        getCompilers: [Object],
        getBlockTransactionCount: [Object],
        getBlockUncleCount: [Object],
        getTransaction: [Object],
        getTransactionFromBlock: [Object],
        getTransactionReceipt: [Object],
        getTransactionCount: [Object],
        call: [Object],
        estimateGas: [Object],
        sendRawTransaction: [Object],
        signTransaction: [Object],
        sendTransaction: [Object],
        sign: [Object],
        compile: [Object],
        submitWork: [Object],
        getWork: [Object],
        coinbase: [Getter],
        getCoinbase: [Object],
        mining: [Getter],
        getMining: [Object],
        hashrate: [Getter],
        getHashrate: [Object],
        syncing: [Getter],
        getSyncing: [Object],
        gasPrice: [Getter],
        getGasPrice: [Object],
        accounts: [Getter],
        getAccounts: [Object],
        blockNumber: [Getter],
        getBlockNumber: [Object],
        protocolVersion: [Getter],
        getProtocolVersion: [Object],
        iban: [Object],
        sendIBANTransaction: [Function: bound transfer] },
     transactionHash: null,
     address: '0xd50fb501d6ef3d04cb415e3c0bb5b053d0a7e95f',
     abi: [ [Object] ],
     say: 
      { [Function: bound ]
        request: [Function: bound ],
        call: [Function: bound ],
        sendTransaction: [Function: bound ],
        estimateGas: [Function: bound ],
        getData: [Function: bound ],
        '': [Circular] },
     allEvents: [Function: bound ] },
  say: 
   { [Function]
     call: [Function],
     sendTransaction: [Function],
     request: [Function: bound ],
     estimateGas: [Function] },
  sendTransaction: [Function],
  send: [Function],
  allEvents: [Function: bound ],
  address: '0xd50fb501d6ef3d04cb415e3c0bb5b053d0a7e95f',
  transactionHash: null }
truffle(dev)> helloworld.say()
'hello world'
truffle(dev)> 
```