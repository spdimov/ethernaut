## Challenge
~~~
The goal of this level is for you to claim ownership of the instance you are given.

  Things that might help

- Look into Solidity's documentation on the `delegatecall` low level function, how it works, how it can be used to delegate operations to on-chain libraries, and what implications it has on execution scope.
- Fallback methods
- Method ids
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Delegate {
    address public owner;

    constructor(address _owner) {
        owner = _owner;
    }

    function pwn() public {
        owner = msg.sender;
    }
}

contract Delegation {
    address public owner;
    Delegate delegate;

    constructor(address _delegateAddress) {
        delegate = Delegate(_delegateAddress);
        owner = msg.sender;
    }

    fallback() external {
        (bool result,) = address(delegate).delegatecall(msg.data);
        if (result) {
            this;
        }
    }
}
```
`

## Solution

```
contract Delegate {
    address public owner;
    
    constructor(address _owner) {
        owner = _owner;
    }

    function attack() public {
        owner = msg.sender;
    }
}
```

To understand this challenge we need to understand two things - how delegatecall works and how state variables are saved in storage. `Delegatecall` function calls another contract code within the context of the current contract i.e. storage is the same aswell as msg.sender and msg.value are not changed. Storage is like a big array of values 32bytes size (there are things like grouping variables at the same position if their sizes are lower than 16bytes both). Each array element is called slot and variables are stored in the way they are declared - in our challenge and malicious contract we would have slot 0 to contain the `owner` variable. Thus, if we change the value of owner in the `Delegate` contract during a `delegatecall` we would actually change the `owner` value in `Delegation` which solves the challenge.


## Key takeaways and useful resources

[Medium article about delegatecall](https://medium.com/@ajaotosinserah/mastering-delegatecall-in-solidity-a-comprehensive-guide-with-evm-walkthrough-6ddf027175c7)
[State variables in storage](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html)
