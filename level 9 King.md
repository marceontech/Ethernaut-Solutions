# Level 9: King

## Challenge Overview

The `King` challenge demonstrates how contracts can be disrupted by forcing them to fail during Ether transfers.  
The goal is to prevent others from becoming the `king` by exploiting the transfer logic.

## Goal

- Deploy a contract that prevents the `King` contract from transferring Ether to it, thus locking the game.

## Solution

The `King` contract makes a transfer to the new king:

```solidity
payable(king).transfer(msg.value);
king = msg.sender;
```

To exploit this:

### 1. Compile `KingAttack.sol` with Solidity ≥0.6.0.

### 2. Deploy & Run

- **Environment** → Injected Web3 (point MetaMask at Sepolia).
- **Contract** → `KingAttack`.
- Click **Deploy** → confirm in MetaMask.

```solidity
// In Ethernaut console, run:
await getBalance(level) // e.g. "1000000000000000000" (1 ETH)
// Then bid = that value + 1 → "1000000000000000001" wei
```

### 3. Under Deployed Contracts → `KingAttack`

- In `claimThrone(address _kingContract)`, **paste your King level’s address**.
- **Do not change Value** (it remains your bid).
- Click **transact** → confirm MetaMask.

Now your `KingAttack` is the king — and any future `king.transfer(...)` will revert inside its fallback.

## Verification

Back in the **Ethernaut** console:

```javascript
(await contract._king()) === "<Your_KingAttack_Address>"
```

---

## Lessons Learned

- A malicious recipient can revert and break your logic.
- Let users `withdraw` refunds rather than forcing them in your critical path.
- Always check the return value of low-level calls (`(bool ok, ) = … call{…}("")`) and decide how to proceed if they fail.
- Don’t assume recipients will always accept ETH.
