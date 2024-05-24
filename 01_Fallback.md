## Challenge
~~~
Look carefully at the contract's code below.
You will beat this level if

1. you claim ownership of the contract
2. you reduce its balance to 0

  Things that might help

- How to send ether when interacting with an ABI
- How to send ether outside of the ABI
- Converting to and from wei/ether units (see `help()` command)
- Fallback methods
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Fallback {
    mapping(address => uint256) public contributions;
    address public owner;

    constructor() {
        owner = msg.sender;
        contributions[msg.sender] = 1000 * (1 ether);
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "caller is not the owner");
        _;
    }

    function contribute() public payable {
        require(msg.value < 0.001 ether);
        contributions[msg.sender] += msg.value;
        if (contributions[msg.sender] > contributions[owner]) {
            owner = msg.sender;
        }
    }

    function getContribution() public view returns (uint256) {
        return contributions[msg.sender];
    }

    function withdraw() public onlyOwner {
        payable(owner).transfer(address(this).balance);
    }

    receive() external payable {
        require(msg.value > 0 && contributions[msg.sender] > 0);
        owner = msg.sender;
    }
}
```
`

## Solution

There are two places in this contract where the owner can be changed - `contribute()` and `receive()`. In order to change it from `contribute()` we need to either send under `0.001 ether` and send over `contributions[owner]` which is initially `1000 ether`. In order to change the owner we should take a look at the `receive()` function. [How does it work?](https://ethereum.stackexchange.com/questions/81994/what-is-the-receive-keyword-in-solidity)

In order to satisfy the required statement in the `receive()` function we should send some ether and to have already called the `contribute()` function with some ether also. After successfully executing the `receive()`, we are now the new owner of the contract which gives is the pass to call the withdraw function and drain the contract funds.

## Key takeaways and useful resources
[ETH Converter](https://eth-converter.com/)
[web3js send transaction](https://web3js.readthedocs.io/en/v1.2.0/web3-eth.html?highlight=sendtransaction#sendtransaction)

![[Pasted image 20240503153312.png]]

