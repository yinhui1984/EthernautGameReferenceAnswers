# 13 Privacy

## 思路

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Privacy {

  bool public locked = true;
  uint256 public ID = block.timestamp;
  uint8 private flattening = 10;
  uint8 private denomination = 255;
  uint16 private awkwardness = uint16(block.timestamp);
  bytes32[3] private data;

  constructor(bytes32[3] memory _data) {
    data = _data;
  }
  
  function unlock(bytes16 _key) public {
    require(_key == bytes16(data[2]));
    locked = false;
  }
}
```

目标, 调用 `unlock`函数将`locked`设置为`flase`

需要得到`bytes16(data[2])`, 嗯, `data`是在构造器中传入的, 然后明文存储的~~

方案: 反编译

```
	WEB3_PROVIDER_URI=http://localhost:8545 panoramix <address_of_contract>
```

```solidity
# Palkeoramix decompiler.

def storage:
  locked is uint8 at storage 0
  ID is uint256 at storage 1
  stor5 is uint256 at storage 5  # <== here

def ID() payable:
  return ID

def locked() payable:
  return bool(locked)

#
#  Regular functions
#

def _fallback() payable: # default function
  revert

def unlock(bytes16 _param1) payable:
  require calldata.size - 4 >=′ 32
  require _param1 == Mask(128, 128, _param1)
  require Mask(128, 128, _param1) == Mask(128, 128, stor5) # <== here
  locked = 0

```

可以看出, 我们需要数据存储在slot5, 获取该数据, 并取其中的bytes16即可

```js
let data = await web3.eth.getStorageAt(instance, 5);
let bytes16 = "0x" + data.slice(2, 34);
await contract.unlock(bytes16);
```



### 参考文章

https://medium.com/@dariusdev/how-to-read-ethereum-contract-storage-44252c8af925

### 参考答案

```
^^.js:45 ˁ(⦿ᴥ⦿)ˀ 正在从关卡获得新的实例... <  < <<请稍等>> >  >
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 19:40:40.489
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xcd475ae5d22e9b9982cde2408cbbe39bb5b343e003a99a6166775d97ffd63581
^^.js:35 => 实例地址0x456B975BD27D793B4Ff7218471b413B3F9270280
redux-logger.js:1  action LOAD_LEVEL_INSTANCE @ 19:40:47.920
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xcd475ae5d22e9b9982cde2408cbbe39bb5b343e003a99a6166775d97ffd63581
redux-logger.js:1  action SET_BLOCK_NUM @ 19:40:55.362
(await contract.locked()).toString();
"true"
let data = await web3.eth.getStorageAt(instance, 5);
undefined
data
"0x5c0273b797cb72ff2f377a5ccccbdb4d763e3862dbb52a996e7c52bd8a496d92"
let bytes16 = "0x" + data.slice(2, 34);
undefined
bytes16
"0x5c0273b797cb72ff2f377a5ccccbdb4d"
await contract.unlock(bytes16);
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x4caeccf89b338e77717a2914b6f8b4ab2e3f6139557232c8a01846f1f7f8de1b
{tx: "0x4caeccf89b338e77717a2914b6f8b4ab2e3f6139557232c8a01846f1f7f8de1b", receipt: {…}, logs: Array(0)}
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x4caeccf89b338e77717a2914b6f8b4ab2e3f6139557232c8a01846f1f7f8de1b
redux-logger.js:1  action SET_BLOCK_NUM @ 19:44:35.359
(await contract.locked()).toString();
"false"
^^.js:45 ⊂(✰‿✰)つ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 19:44:43.740
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0xf90db136e9eb0125ed4b1c9ab1f85408808bb7f3d9efac77aee516b359e7febd
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 19:44:50.954
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0xf90db136e9eb0125ed4b1c9ab1f85408808bb7f3d9efac77aee516b359e7febd
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:73 |[●▪▪●]| 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 19:44:55.363

```



