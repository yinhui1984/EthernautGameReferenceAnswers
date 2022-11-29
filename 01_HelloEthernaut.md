# 01_HelloEthernaut

## 环境准备

### 安装MataMask

略

### 搭建本地ethernaut

可以使用在线版本 https://ethernaut.openzeppelin.com/

在线ethernaut使用的eth测试网一个是慢, 第二是测试币获取太少, 多玩几次就不够了, 所以在本地搭建比较好

克隆https://github.com/OpenZeppelin/ethernaut 到本地

```
git clone https://github.com/OpenZeppelin/ethernaut.git

cd ethernaut

npm i

启动Ganache, 导入私钥到MetaMask (也可以运行npm run network 运行hardhat本地节点)

npm run compile:contracts

将client/src/constants.js 中的ACTIVE_NETWORK 设置为 NETWORKS.LOCAL

将client/scripts/deploy_contracts.mjs 中的 deployContracts 的 gas 改为 25000000 (小于30000000的值)

npm run deploy:contracts

确保npm使用16版本: nvm use 16
然后运行: npm run start:ethernaut

```



## 参考答案

点击页面上的create new instance, 打开chrome的console:

```log
^^.js:45 ≧◔◡◔≦ Requesting new instance from level... <  < <<PLEASE WAIT>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 11:47:24.442
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x37aaa4202b97404d1e65cc6787c6d851f67d26b62931e10314edaebfcca33ebf
^^.js:35 => Instance address0x5d424b534F9c23A70990CB330a922C77947c97Db
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 11:47:32.174
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x37aaa4202b97404d1e65cc6787c6d851f67d26b62931e10314edaebfcca33ebf
redux-logger.js:1  action SET_BLOCK_NUM @ 11:47:38.012
contract
TruffleContract {methods: {…}, abi: Array(11), address: "0x5d424b534F9c23A70990CB330a922C77947c97Db", transactionHash: undefined, constructor: ƒ, …}
await contract.info()
"You will find what you need in info1()."
await contract.info1()
"Try info2(), but with \"hello\" as a parameter."
await contract.info2("hello")
"The property infoNum holds the number of the next info method to call."
await contract.infoNum()
BN {negative: 0, words: Array(2), length: 1, red: null}length: 1negative: 0red: nullwords: Array(2)0: 42length: 2__proto__: Array(0)__proto__: Object
await contract.info42()
"theMethodName is the name of the next method."
await contract.theMethodName()
"The method name is method7123949."
await contract.method7123949()
"If you know the password, submit it to authenticate()."
await contract.password()
"ethernaut0"
await contract.authenticate("ethernaut0")
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xd3a3ea70ca6d93e9ab92a6b054e9c86a6a818a33c7bb1ace0670e98c2eca2932
{tx: "0xd3a3ea70ca6d93e9ab92a6b054e9c86a6a818a33c7bb1ace0670e98c2eca2932", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xd3a3ea70ca6d93e9ab92a6b054e9c86a6a818a33c7bb1ace0670e98c2eca2932
redux-logger.js:1  action SET_BLOCK_NUM @ 11:49:48.014
^^.js:45 龴ↀ◡ↀ龴 Submitting level instance... <  < <<PLEASE WAIT>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 11:49:59.478
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xce7b2908a464bc855fc8cfc7ec9ce705454c024652e7d67af0208438c8098cf6
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 11:50:13.145
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xce7b2908a464bc855fc8cfc7ec9ce705454c024652e7d67af0208438c8098cf6
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:73 ヽ(￣(ｴ)￣)ﾉ Well done, You have completed this level!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 11:50:18.041

```



##合约

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Instance {

  string public password;
  uint8 public infoNum = 42;
  string public theMethodName = 'The method name is method7123949.';
  bool private cleared = false;

  // constructor
  constructor(string memory _password) {
    password = _password;
  }

  function info() public pure returns (string memory) {
    return 'You will find what you need in info1().';
  }

  function info1() public pure returns (string memory) {
    return 'Try info2(), but with "hello" as a parameter.';
  }

  function info2(string memory param) public pure returns (string memory) {
    if(keccak256(abi.encodePacked(param)) == keccak256(abi.encodePacked('hello'))) {
      return 'The property infoNum holds the number of the next info method to call.';
    }
    return 'Wrong parameter.';
  }

  function info42() public pure returns (string memory) {
    return 'theMethodName is the name of the next method.';
  }

  function method7123949() public pure returns (string memory) {
    return 'If you know the password, submit it to authenticate().';
  }

  function authenticate(string memory passkey) public {
    if(keccak256(abi.encodePacked(passkey)) == keccak256(abi.encodePacked(password))) {
      cleared = true;
    }
  }

  function getCleared() public view returns (bool) {
    return cleared;
  }
}
```



## 知识点

### js调用合约函数

```js
await contract.theMethodName(theArg)
```



### js访问合约公开属性

```js
await contract.thePropertyName()
```



### solidity比较字符串

```solidity
if(keccak256(abi.encodePacked(str1)) == keccak256(abi.encodePacked(str2))) {
}
```

