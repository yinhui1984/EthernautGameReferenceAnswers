# 27 DoubleEntryPoint

## 思路

目标: 找出合约中的bug, 该bug会导致CryptoVault中的DEP token被划走, 写一个detectionbot阻止漏洞

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "openzeppelin-contracts-08/access/Ownable.sol";
import "openzeppelin-contracts-08/token/ERC20/ERC20.sol";

interface DelegateERC20 {
  function delegateTransfer(address to, uint256 value, address origSender) external returns (bool);
}

interface IDetectionBot {
    function handleTransaction(address user, bytes calldata msgData) external;
}

interface IForta {
    function setDetectionBot(address detectionBotAddress) external;
    function notify(address user, bytes calldata msgData) external;
    function raiseAlert(address user) external;
}

contract Forta is IForta {
  mapping(address => IDetectionBot) public usersDetectionBots;
  mapping(address => uint256) public botRaisedAlerts;

  function setDetectionBot(address detectionBotAddress) external override {
      usersDetectionBots[msg.sender] = IDetectionBot(detectionBotAddress);
  }

  function notify(address user, bytes calldata msgData) external override {
    if(address(usersDetectionBots[user]) == address(0)) return;
    try usersDetectionBots[user].handleTransaction(user, msgData) {
        return;
    } catch {}
  }

  function raiseAlert(address user) external override {
      if(address(usersDetectionBots[user]) != msg.sender) return;
      botRaisedAlerts[msg.sender] += 1;
  } 
}

contract CryptoVault {
    address public sweptTokensRecipient;
    IERC20 public underlying;

    constructor(address recipient) {
        sweptTokensRecipient = recipient;
    }

    function setUnderlying(address latestToken) public {
        require(address(underlying) == address(0), "Already set");
        underlying = IERC20(latestToken);
    }

    /*
    ...
    */

    function sweepToken(IERC20 token) public {
        require(token != underlying, "Can't transfer underlying token");
        token.transfer(sweptTokensRecipient, token.balanceOf(address(this)));
    }
}

contract LegacyToken is ERC20("LegacyToken", "LGT"), Ownable {
    DelegateERC20 public delegate;

    function mint(address to, uint256 amount) public onlyOwner {
        _mint(to, amount);
    }

    function delegateToNewContract(DelegateERC20 newContract) public onlyOwner {
        delegate = newContract;
    }

    function transfer(address to, uint256 value) public override returns (bool) {
        if (address(delegate) == address(0)) {
            return super.transfer(to, value);
        } else {
            return delegate.delegateTransfer(to, value, msg.sender);
        }
    }
}

contract DoubleEntryPoint is ERC20("DoubleEntryPointToken", "DET"), DelegateERC20, Ownable {
    address public cryptoVault;
    address public player;
    address public delegatedFrom;
    Forta public forta;

    constructor(address legacyToken, address vaultAddress, address fortaAddress, address playerAddress) {
        delegatedFrom = legacyToken;
        forta = Forta(fortaAddress);
        player = playerAddress;
        cryptoVault = vaultAddress;
        _mint(cryptoVault, 100 ether);
    }

    modifier onlyDelegateFrom() {
        require(msg.sender == delegatedFrom, "Not legacy contract");
        _;
    }

    modifier fortaNotify() {
        address detectionBot = address(forta.usersDetectionBots(player));

        // Cache old number of bot alerts
        uint256 previousValue = forta.botRaisedAlerts(detectionBot);

        // Notify Forta
        forta.notify(player, msg.data);

        // Continue execution
        _;

        // Check if alarms have been raised
        if(forta.botRaisedAlerts(detectionBot) > previousValue) revert("Alert has been triggered, reverting");
    }

    function delegateTransfer(
        address to,
        uint256 value,
        address origSender
    ) public override onlyDelegateFrom fortaNotify returns (bool) {
        _transfer(origSender, to, value);
        return true;
    }
}
```

思路:

上面的合约和变量很多, 并且我们不清楚题目初始化实例的时候将各个状态变量设置成什么值了, 所以我建了一个forge项目, 将他们都打印出来了

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

import "forge-std/Script.sol";
import "../src/DoubleEntryPoint.sol";

contract Hacker is Script {
    function setUp() public {}

    function run() public {
        //vm.broadcast();
        console2.log("tx.origin:", tx.origin);
        console2.log("msg.sender", msg.sender);

        address instance = 0xee9F9420811cdE418797f902A48E1406b4704f98;
        DoubleEntryPoint dep = DoubleEntryPoint(instance);

        Forta forta = dep.forta();
        CryptoVault vault = CryptoVault(dep.cryptoVault());
        address sweptTokensRecipient = vault.sweptTokensRecipient();
        IERC20 underlying = vault.underlying();
        address player = dep.player();
        address delegateFrom = dep.delegatedFrom();
        console2.log("vault:", address(vault));
        console2.log("player:", player);
        console2.log("forta:", address(forta));
        console2.log("sweptTokensRecipient:", sweptTokensRecipient);
        console2.log("delegateFrom:", delegateFrom);
        console2.log(
            "delegateFrom token name:",
            ERC20(address(delegateFrom)).name()
        );
        console2.log("underlying:", address(underlying));
        console2.log(
            "underlying token name:",
            ERC20(address(underlying)).name()
        );

        LegacyToken LegacyToken_delegateFrom = LegacyToken(
            address(delegateFrom)
        );

        DelegateERC20 _delegate = LegacyToken_delegateFrom.delegate();
        console2.log("delegate in legacy:", address(_delegate));

        DoubleEntryPoint DoubleEntryPoint_underlying = DoubleEntryPoint(
            address(underlying)
        );

        console2.log(
            "LegacyToken balance of vault:",
            LegacyToken_delegateFrom.balanceOf(address(vault))
        );

        console2.log(
            "DoubleEntryPoint balance of vault:",
            DoubleEntryPoint_underlying.balanceOf(address(vault))
        );

        console2.log(
            "LegacyToken balance of player:",
            LegacyToken_delegateFrom.balanceOf(address(player))
        );

        console2.log(
            "DoubleEntryPoint balance of player:",
            DoubleEntryPoint_underlying.balanceOf(address(player))
        );

        console2.log("----------------------------------");
        //!!!
        vault.sweepToken(LegacyToken_delegateFrom);

        console2.log(
            "LegacyToken balance of vault:",
            LegacyToken_delegateFrom.balanceOf(address(vault))
        );

        console2.log(
            "DoubleEntryPoint balance of vault:",
            DoubleEntryPoint_underlying.balanceOf(address(vault))
        );

        console2.log(
            "LegacyToken balance of player:",
            LegacyToken_delegateFrom.balanceOf(address(player))
        );

        console2.log(
            "DoubleEntryPoint balance of player:",
            DoubleEntryPoint_underlying.balanceOf(address(player))
        );
    }
}

```



并尝试使用DoubleEntryPoint token 和 legacyToken 调用 `vault.sweepToken`

前者会报错: Can't transfer underlying token

后者成功了

(源代码包: https://github.com/yinhui1984/EthernautGameReferenceAnswers/blob/main/DoubleEntryPointDemo.tar.gz )

```
== Logs ==
  tx.origin: 0x37bd42f4cF5785F5A36244e4800636cE5787d5e8
  msg.sender 0x37bd42f4cF5785F5A36244e4800636cE5787d5e8
  vault: 0x2EF02607471D9A1D27Ea0F94633EB39e079f39C5
  player: 0x37bd42f4cF5785F5A36244e4800636cE5787d5e8
  forta: 0xd0e5b6aACA73516bBD5EaDcaE3f1301636528A4C
  sweptTokensRecipient: 0x37bd42f4cF5785F5A36244e4800636cE5787d5e8
  delegateFrom: 0x069d917d3377beE69F68390a7c89d9c334e515b8
  delegateFrom token name: LegacyToken
  underlying: 0xee9F9420811cdE418797f902A48E1406b4704f98
  underlying token name: DoubleEntryPointToken
  delegate in legacy: 0xee9F9420811cdE418797f902A48E1406b4704f98
  LegacyToken balance of vault: 100000000000000000000
  DoubleEntryPoint balance of vault: 100000000000000000000
  LegacyToken balance of player: 0
  DoubleEntryPoint balance of player: 0
  ----------------------------------
  LegacyToken balance of vault: 100000000000000000000
  DoubleEntryPoint balance of vault: 0
  LegacyToken balance of player: 0
  DoubleEntryPoint balance of player: 100000000000000000000
```

需要写一个机器人阻止该操作,  机器人使用是一个"中间人攻击"模式, 发现此类操作的时候, revert并报警

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IDetectionBot {
    function handleTransaction(address user, bytes calldata msgData) external;
}

interface IForta {
    function setDetectionBot(address detectionBotAddress) external;
    function raiseAlert(address user) external;
}

interface DoubleEntryPoint{
    function forta() external view returns (address ) ;
    function cryptoVault() external view returns (address) ;
}

contract MyDetectionBot is IDetectionBot {

    DoubleEntryPoint private dep;

    constructor(address _doubleEntryPoint){
        dep = DoubleEntryPoint(
            _doubleEntryPoint
        );
    }

    function handleTransaction(address user, bytes calldata msgData) external {
    
        //function delegateTransfer(address to, uint256 value,address origSender)

        (, , address addr) = abi.decode(
            msgData[4:],
            (address, uint256, address)
        );
        if (addr == dep.cryptoVault()) {
            IForta(dep.forta()).raiseAlert(user);
        }
    }
}


```



注意: 注册机器人的时候, 机器人是和`msg.sender`相关联的, 

```
  function setDetectionBot(address detectionBotAddress) external override {
      usersDetectionBots[msg.sender] = IDetectionBot(detectionBotAddress);
  }
```

为了报警的时候对象是player, 所以我们使用web3.js来调用注册函数进行机器人注册(这样保证msg.sender是player, 否则提交题目实例的时候会闯关失败)

假设MyDetectionBot部署后的地址是`0x987bA5791848c789B9748afED79445CEACAC85db`

```js
calldata = web3.eth.abi.encodeFunctionCall({
    name: 'setDetectionBot',
    type: 'function',
    inputs: [{
        type: 'address',
        name: 'detectionBotAddress'
    }]
}, ['0x987bA5791848c789B9748afED79445CEACAC85db']);

forta = await contract.forta()

await web3.eth.sendTransaction(
	{
		from: player, 
		to: forta,
		data:calldata
	}
)
```

注册成功后, 我们在运行forge项目中的测试脚本, 可以从日志从看到, 操作被阻止了

```
== Logs ==
  tx.origin: 0x37bd42f4cF5785F5A36244e4800636cE5787d5e8
  msg.sender 0x37bd42f4cF5785F5A36244e4800636cE5787d5e8
  vault: 0x2EF02607471D9A1D27Ea0F94633EB39e079f39C5
  player: 0x37bd42f4cF5785F5A36244e4800636cE5787d5e8
  forta: 0xd0e5b6aACA73516bBD5EaDcaE3f1301636528A4C
  sweptTokensRecipient: 0x37bd42f4cF5785F5A36244e4800636cE5787d5e8
  delegateFrom: 0x069d917d3377beE69F68390a7c89d9c334e515b8
  delegateFrom token name: LegacyToken
  underlying: 0xee9F9420811cdE418797f902A48E1406b4704f98
  underlying token name: DoubleEntryPointToken
  delegate in legacy: 0xee9F9420811cdE418797f902A48E1406b4704f98
  LegacyToken balance of vault: 100000000000000000000
  DoubleEntryPoint balance of vault: 100000000000000000000
  LegacyToken balance of player: 0
  DoubleEntryPoint balance of player: 0
  ----------------------------------
Error: 
Alert has been triggered, reverting
```

