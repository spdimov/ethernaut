## Challenge
~~~
Claim ownership of the contract below to complete this level.

  Things that might help

- Solidity Remix IDE
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "openzeppelin-contracts-06/math/SafeMath.sol";

contract Fallout {
    using SafeMath for uint256;

    mapping(address => uint256) allocations;
    address payable public owner;

    /* constructor */
    function Fal1out() public payable {
        owner = msg.sender;
        allocations[owner] = msg.value;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "caller is not the owner");
        _;
    }

    function allocate() public payable {
        allocations[msg.sender] = allocations[msg.sender].add(msg.value);
    }

    function sendAllocation(address payable allocator) public {
        require(allocations[allocator] > 0);
        allocator.transfer(allocations[allocator]);
    }

    function collectAllocations() public onlyOwner {
        msg.sender.transfer(address(this).balance);
    }

    function allocatorBalance(address allocator) public view returns (uint256) {
        return allocations[allocator];
    }
}
```


## Solution

In this contract we can see that `Fal1out()` function is not guarded with the onlyOwner modifier thus allowing us to call it at any time. Before Solidity version 0.4.22, the only way of defining a constructor was to create a function with the same name as the contract class containing.


## Key takeaways and useful resources

[Incorrect constructor name](https://swcregistry.io/docs/SWC-118/)

