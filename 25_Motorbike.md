## Challenge
~~~
Ethernaut's motorbike has a brand new upgradeable engine design.

Would you be able to `selfdestruct` its engine and make the motorbike unusable ?

Things that might help:

- [EIP-1967](https://eips.ethereum.org/EIPS/eip-1967)
- [UUPS](https://forum.openzeppelin.com/t/uups-proxies-tutorial-solidity-javascript/7786) upgradeable pattern
- [Initializable](https://github.com/OpenZeppelin/openzeppelin-upgrades/blob/master/packages/core/contracts/Initializable.sol) contract
~~~
## Contract
```
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
        (bool success,) = _logic.delegatecall(abi.encodeWithSignature("initialize()"));
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
    fallback() external payable virtual {
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
    function _upgradeToAndCall(address newImplementation, bytes memory data) internal {
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
`

## Solution

Will try the GTDA(Goals, Tags, Diagram, Attack) method described by Owen Thurm

Goals:
- Understand what the contracts do	
	- Motorbike is used as a proxy contract which keeps the storage slot of the implementation contract at storage slot 0
		- has a constructor that accepts logic address
			- inside we have a basic check if the address is a contract address
			- then we set the implementation contract address to the provided address
			- delegatecall with `abi.encodeWithSignature("initialize()")`
			-  check if success
		- basic delegate function, basic fallback function, basic function and struct to directly access storage slots
	- Engine is the implementation contract for Motorbike
		- implementation slot on slot 0
		- two variables on slot 1 and 2 - `address upgrader` and `uint256 horsePower`
		- `initialize()` function to set the both variables, horsePower to 1000 and upgrader to msg.sender
		- `upgradeToAndCall()`
			- checks if the msg.sender == upgrader(initially the deployer of the proxy contract)
			- if check is successful, new implementation is set and the provided data is sent to that address via `delegatecall`
			- `_setImeplementation()` 
	
- Go through the possible flows
	We have only to functions which are not internal or private `initialize()` and `upgradeToAndCall()`. `initialize()` has already been executed in the context of Motorbike. Also the `upgradeToAndCall()` function has check `msg.sender == upgrader` so we wont be able to execute it in the context of Motorbike. The only possible flow is to use these functions directly on the implementation contract. We can become the `upgrader` of the implementation contract by calling the initialize function, but first we need the address of the implementation contract.
	`web3.eth.getStorageAt(<proxy_contract_address>,<storage_slot>)` = address of the impl contract
	  
	From here we can run `initialize()` and become the upgrader, which allows us to run updateAndCall()
	NOTE: Not working because of EIP-6780
	```
	contract Hack {
    function pwn(IEngine target) external {
        target.initialize();
        target.upgradeToAndCall(address(this), abi.encodeWithSelector(this.kill.selector));
    }

    function kill() external {
        selfdestruct(payable(address(0)));
    }
}
	```

	
## Key takeaways and useful resources

[Ethereum blockchain developer articles + video](https://ethereum-blockchain-developer.com/110-upgrade-smart-contracts/00-project/)
[Owen Thurm's video](https://www.youtube.com/watch?v=e5lWvt1rIm0)
[Open Zeppelin Upgrade pattern](https://docs.openzeppelin.com/upgrades-plugins/1.x/proxies)
EIP-897, 1822, 1967, 1538, 2535 on https://eips.ethereum.org/all
