## Challenge
~~~
The goal of this level is for you to hack the basic token contract below.

You are given 20 tokens to start with and you will beat the level if you somehow manage to get your hands on any additional tokens. Preferably a very large amount of tokens.

  Things that might help:

- What is an odometer?
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

contract Token {
    mapping(address => uint256) balances;
    uint256 public totalSupply;

    constructor(uint256 _initialSupply) public {
        balances[msg.sender] = totalSupply = _initialSupply;
    }

    function transfer(address _to, uint256 _value) public returns (bool) {
        require(balances[msg.sender] - _value >= 0);
        balances[msg.sender] -= _value;
        balances[_to] += _value;
        return true;
    }

    function balanceOf(address _owner) public view returns (uint256 balance) {
        return balances[_owner];
    }
}
```
`

## Solution

Since our initial balance would be 0, we simply have to call the msg.sender with `_value` greater than 0. Why is that? Because of the underlying underflow and overflow implementation of solidity.
If we try to add 1 to `type(uint256).max` (this is the greatest value which uint256 can hold `2^256 -1`) we would get a result of 0. The same is applicable when we try to subtract 1 from `type(uint256).min` - we would get as a result 2^256 -1 

## Key takeaways and useful resources

[hackernoon article](https://hackernoon.com/hack-solidity-integer-overflow-and-underflow)
[OZ SafeMath library](https://github.com/ConsenSysMesh/openzeppelin-solidity/blob/master/contracts/math/SafeMath.sol)

