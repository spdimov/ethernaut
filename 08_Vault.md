## Challenge
~~~
Unlock the vault to pass the level!
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Vault {
    bool public locked;
    bytes32 private password;

    constructor(bytes32 _password) {
        locked = true;
        password = _password;
    }

    function unlock(bytes32 _password) public {
        if (password == _password) {
            locked = false;
        }
    }
}
```
`

## Solution

Private variables does not mean they cannot be read, it means that they cannot be directly access by other contracts. All state variables of contracts can be seen as they are kept in the storage of the account. One way to do this is via the web3js library. In the Delegation challenge, we have discussed the layout of state variables in storage.

`web3.eth.getStorageAt(addressHexString, position)` and in our case the position would be 0
## Key takeaways and useful resources

[Stack Exchange about private variables](https://ethereum.stackexchange.com/questions/79603/what-are-private-variables-in-solidity)


