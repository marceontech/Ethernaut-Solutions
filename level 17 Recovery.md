# Level 17: Recovery

## Challenge Overview

This challenge illustrates the importance of understanding contract destruction and the recovery of funds sent to destroyed contracts.  
The goal is to identify and recover Ether held by a destroyed contract.



## Solution

The `Recovery` contract holds newly created `SimpleToken` instances and has no withdrawal function,  
so any tokens sent to Recovery appear stuck. To recover them, you don’t need to compute a destroyed-contract address or use `selfdestruct` on Ether—this challenge is about **ERC-20 tokens**.



## How to Exploit

### 1. Find the Token’s Address

Whenever you called `generateToken("Name","SYM")` on Recovery, it deployed a `SimpleToken` and saved its address in the public array `tokenContracts`.  
To recover tokens from the first generated token, do:

```javascript
// 'contract' is the Recovery instance
const tokenAddr = await contract.tokenContracts(0);
console.log("Token address:", tokenAddr);
```



### 2. See How Many Tokens Recovery Holds

```javascript
const stuck = await web3.eth.call({
  to: tokenAddr,
  data: web3.eth.abi.encodeFunctionSignature("balanceOf(address)") +
        web3.eth.abi.encodeParameter("address", contract.address).slice(2)
});
console.log("Recovery holds:", web3.utils.toBN(stuck).toString());
// e.g. "1000000000000000000000" (1,000 tokens * 10^18)
```



### 3. Deploy a Helper That “Acts as Recovery” to Call `transfer`

When `SimpleToken.transfer(...)` runs, `msg.sender` must be the address holding tokens — in this case, the **Recovery** contract.  
By having Recovery itself deploy a tiny helper whose constructor calls `token.transfer(player, amount)`,  
you ensure `msg.sender == recovery.address`.

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

interface ISimpleToken {
    function transfer(address to, uint256 amount) external;
}

contract RecoveryHelper {
    constructor(address tokenAddr, address to, uint256 amount) {
        // This runs with msg.sender == Recovery
        ISimpleToken(tokenAddr).transfer(to, amount);
        selfdestruct(payable(msg.sender)); // optionally remove helper
    }
}
```



### 4. Have Recovery Deploy `RecoveryHelper` in One Shot

In the Ethernaut console, replace the usual `generateToken` call’s bytecode with `RecoveryHelper`'s and have Recovery create it:

```javascript
// Compile RecoveryHelper in Remix, copy its deploy bytecode:
const helperBytecode = "<...RecoveryHelper bytecode...>";

// Pack generateToken’s selector + dummy args:
const selector = web3.eth.abi.encodeFunctionSignature("generateToken(string,string)");
const dummyData = selector +
  web3.eth.abi.encodeParameter("string","x").slice(2) +
  web3.eth.abi.encodeParameter("string","y").slice(2);

// Overwrite the bytecode segment so Recovery deploys RecoveryHelper instead:
const payload = dummyData.replace(
  /^8x[0-9a-fA-F]{8}[0-9a-fA-F]+/,
  helperBytecode
);

// Send the transaction so Recovery’s new CREATE runs RecoveryHelper’s constructor:
// Pass (tokenAddr, player, stuck) as the arguments in order
await sendTransaction({
  from: player,
  to: contract.address,
  data: payload
});
```



### 5. Verify & Submit in Ethernaut Console

```javascript
const final = await web3.eth.call({
  to: tokenAddr,
  data: web3.eth.abi.encodeFunctionSignature("balanceOf(address)") +
        web3.eth.abi.encodeParameter("address", player).slice(2)
});
console.log("Player now has:", web3.utils.toBN(final).toString()); // should equal stuck
```



## Lessons Learned

- This level is about recovering **tokens**, not Ether—don’t confuse `SELFDESTRUCT` tricks with ERC-20 logic.
- You can simply use `tokenContracts(index)` to look up each token’s address—you don’t need to calculate it manually.
- Only the current holder can call `transfer(...)`. By having Recovery create a helper contract, we make Recovery itself call `transfer`, so it sends the stuck tokens back.
- If your contract accepts tokens but has no way to send them out, a poorly designed contract may “lock” user tokens forever.
