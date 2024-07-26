---
title: learning solidity
tags:
  - Blockchain
  - Ethereum
category:
  - Blockchain
author: bsyonline
lede: 没有摘要
date: 2018-05-05 17:08:17
thumbnail:
---

Solidity 是实现智能合约官方的推荐语言之一。Solidity 的语法和 Javascript 类似。

#### **版本指令**
每个智能合约的代码都是以版本指令声明开始的。这个是一个空的智能合约。
```
pragma solidity ^0.4.23;
contract HelloWorld {
}
```

#### **状态变量和数据类型**
状态变量会被永久地保存在合约中，保存在区块链上。
```
pragma solidity ^0.4.23;
contract HelloWorld {
        uint a = 100;
}
```
上边的例子声明了一个 uint 类型的变量并赋值为 100 。在 Solidity 中 uint 实际上是 uint256 的省略写法。
Solidity 还有其他数据类型：布尔类型 bool，整型 uint，uint8，uint32，uint64 ... uint256，字符串类型 string，数组类型 []，映射 mapping 等。
##### 地址
以太坊区块链由 account (账户)组成。一个帐户的余额是以太（在以太坊区块链上使用的币种），帐户之间可以支付和接受以太币。帐户地址是属于特定用户（或智能合约）的。每个帐户都有一个“地址”，是账户唯一的标识。
msg.sender 是当前调用者（或智能合约）的 address，是一个全局变量，并且它总是存在的。
```
mapping (address => uint) favoriteNumber;

function setMyNumber(uint _myNumber) public {
  // 更新我们的 `favoriteNumber` 映射来将 `_myNumber`存储在 `msg.sender`名下
  favoriteNumber[msg.sender] = _myNumber;
  // 存储数据至映射的方法和将数据存储在数组相似
}

function whatIsMyNumber() public view returns (uint) {
  // 拿到存储在调用者地址名下的值
  // 若调用者还没调用 setMyNumber， 则值为 `0`
  return favoriteNumber[msg.sender];
}
```

##### 映射
映射的定义方式如下：
```
//对于金融应用程序，将用户的余额保存在一个 uint类型的变量中：
mapping (address => uint) public accountBalance;
//或者可以用来通过userId 存储/查找的用户名
mapping (uint => string) userIdToName;
```
映射本质上是存储和查找数据所用的键-值对。在第一个例子中，键是一个 address，值是一个 uint，在第二个例子中，键是一个uint，值是一个 string。


#### **运算**
Solidity 支持基本的数学运算：加，减，乘，除，取模，乘方。
```
pragma solidity ^0.4.23;
contract HelloWorld {
        uint a = 3;
        uint b = 2;
        uint c = a + b;
        uint d = a - b;
        uint e = a * b;
        uint f = a / b;
        uint g = a % b;
        uint h = a ** b;	// a 的 b 次方
        uint i = a ^ b;	// a 的 b 次方
}
```
#### **结构体**
结构体是一种更复杂的数据类型，可以包含多个属性。
```
struct  Person {
        string name;
        uint age;
}
```
#### **数组**
Solidity 支持两种数组：静态数组和动态数组。
```
pragma solidity ^0.4.23;
contract HelloWorld {
        // 固定长度为2的静态数组:
        uint[2] fixedArray;
        // 固定长度为5的string类型的静态数组:
        string[5] stringArray;
        // 动态数组，长度不固定，可以动态添加元素:
        uint[] dynamicArray;
        // 结构体数组
        Person[] structArray;
        // 公共数组
        uint[] public intArray;
        uint[] numbers;
        numbers.push(5);
}
```
Solidity 会为 public 数组自动创建 getter 方法，其他的合约可以从 public 数组中读取数据（不可写），所以可以用 public 数组存储公共数据。

#### **函数**
Solidity 中可以通过一下形式创建函数：
```
function fun(string _arg1, uint _arg2){

}
```
虽然函数的属性默认为公共，但是这会易于受到攻击，所以将函数声明为私有，在需要的时候再改为共有是一个好习惯。用 private 就可以声明一个私有函数。
```
function _fun(uint _number) private {
        numbers.push(_number);
}
```
以（\_）开头是私有函数的命名约定。
##### internal 和 external
除 public 和 private 属性之外，Solidity 还使用了另外两个描述函数可见性的修饰词：internal（内部） 和 external（外部）。internal 和 private 类似，不过， 如果某个合约继承自其父合约，这个合约即可以访问父合约中定义的“内部”函数。external 与 public 类似，只不过这些函数只能在合约之外调用，它们不能被合约内的其他函数调用。
声明函数 internal 或 external 类型的语法，与声明 private 和 public类 型相同：
```
contract Sandwich {
        uint private sandwichesEaten = 0;

        function eat() internal {
                sandwichesEaten++;
        }
}

contract BLT is Sandwich {
        uint private baconSandwichesEaten = 0;

        function eatWithBacon() public returns (string) {
                baconSandwichesEaten++;
                // 因为eat() 是internal 的，所以我们能在这里调用
                eat();
        }
}
```


上边的函数没有返回值，可以通过 returns 定义返回值。

```
function fun() public returns (string) {
        return "abc";
}
```
上边的函数没有改变任何值，所以可以把它定义为 view 。
```
function fun() public view returns (string) {
        return "abc";
}
```
Solidity 里还有一种特殊的函数，pure 函数，即函数不访问应用数据，返回值只依赖于输入参数。
```
function _multiply(uint a, uint b) private pure returns (uint) {
        return a * b;
}
```
#### **类型转换**

两种不同数据类型的数据进行运算时会出现类型转换的问题，有时会出现错误。
```
uint8 a = 5;
uint b = 6;
// 将会抛出错误，因为 a * b 返回 uint, 而不是 uint8
uint8 c = a * b;
```
要么将 c 的类型变成 uint，要么将 b 强转成 uint8 。

```
uint8 c = a * uint8(b);
```

#### **事件**

事件是合约和区块链通讯的一种机制。前端应用“监听”某些事件，并做出反应。

```
event IntegersAdded(uint x, uint y, uint result);
function add(uint _x, uint _y) public {
        uint result = _x + _y;
        //触发事件，通知app
        IntegersAdded(_x, _y, result);
        return result;
}
```
同时在前端 javascript 中监听事件。
```
var MyContract = web3.eth.contract(abi).at(contractAddress)
// 监听 `IntegersAdded` 事件, 并且更新UI
var event = MyContract.add(function(error, result) {
  if (error) return
  updateUI()
})
```

#### **Require**
在调用函数的过程中，经常需要加上一些前置规则的验证，这就要用到 require 。require 使得函数在执行过程中，当不满足某些条件时抛出错误，并停止执行。
```
function sayHiToVitalik(string _name) public returns (string) {
        // 比较 _name 是否等于 "Vitalik". 如果不成立，抛出异常并终止程序
        // (敲黑板: Solidity 并不支持原生的字符串比较, 我们只能通过比较
        // 两字符串的 keccak256 哈希值来进行判断)
        require(keccak256(_name) == keccak256("Vitalik"));
        // 如果返回 true, 运行如下语句
        return "Hi!";
}
```

#### **继承**

 当代合约码过于冗长的时候，最好将代码和逻辑分拆到多个不同的合约中，以便于管理，这就要用到继承。
```
 contract Doge {
       function catchphrase() public returns (string) {
               return "So Wow CryptoDoge";
        }
}
contract BabyDoge is Doge {
        function anotherCatchphrase() public returns (string) {
                return "Such Moon BabyDoge";
        }
}
```
BabyDoge  继承了 Doge ，所以可以访问 Doge 中的所有公有函数。

#### **接口**
Solidity 也支持接口，

#### **import**
通常情况下，当 Solidity 项目中的代码太长的时候我们就把它分成多个文件以便于管理。在 Solidity 中，当你有多个文件并且想把一个文件导入另一个文件时，可以使用 import 语句。
```
import "./someothercontract.sol";

contract newContract is SomeOtherContract {

}
```
./ 就是同一目录的意思。
#### **Storage与Memory**
在 Solidity 中，有两个地方可以存储变量 ： storage 或 memory 。Storage 变量是指永久存储在区块链中的变量。 Memory 变量则是临时的，当外部函数对某合约调用完成时，内存型变量即被移除。 状态变量（在函数之外声明的变量）默认为“storage”形式，并永久写入区块链；而在函数内部声明的变量是“memory”型的，它们函数调用结束后消失。
大多数时候都用不到这些关键字，默认情况下 Solidity 会自动处理它们。然而也有一些情况下，你需要手动声明存储类型，主要用于处理函数内的 结构体和数组时：
```
contract SandwichFactory {
        struct Sandwich {
              string name;
              string status;
        }

        Sandwich[] sandwiches;

        function eatSandwich(uint _index) public {
                 // Sandwich mySandwich = sandwiches[_index];

                 // ^ 看上去很直接，不过 Solidity 将会给出警告
                 // 告诉你应该明确在这里定义 `storage` 或者 `memory`。

                 // 所以你应该明确定义 `storage`:
                 Sandwich storage mySandwich = sandwiches[_index];
       	  // ...这样 `mySandwich` 是指向 `sandwiches[_index]`的指针
    	  // 在存储里，另外...
    	  mySandwich.status = "Eaten!";
    	  // ...这将永久把 `sandwiches[_index]` 变为区块链上的存储

    	  // 如果你只想要一个副本，可以使用`memory`:
    	  Sandwich memory anotherSandwich = sandwiches[_index + 1];
    	  // ...这样 `anotherSandwich` 就仅仅是一个内存里的副本了
    	  // 另外
   	  anotherSandwich.status = "Eaten!";
   	  // ...将仅仅修改临时变量，对 `sandwiches[_index + 1]` 没有任何影响
   	  // 不过你可以这样做:
    	  sandwiches[_index + 1] = anotherSandwich;
   	  // ...如果你想把副本的改动保存回区块链存储
        }
}
```
