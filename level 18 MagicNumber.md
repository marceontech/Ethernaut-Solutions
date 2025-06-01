# Level 18: MagicNumber

## Challenge Overview

This challenge focuses on deploying minimal bytecode to satisfy a specific condition.  
The goal is to deploy a contract that returns the magic number 42 when called.

## Solution

You need to deploy a 10-byte contract that always returns 42 (`0x2a`) on any call.  
Below is a concise, corrected version.



## 1. Minimal Runtime Bytecode

At the EVM opcode level, returning a constant 32-byte value (42) requires exactly these steps:

```
602a       // PUSH1 0x2a        ← push 42 onto the stack  
6000       // PUSH1 0x00        ← memory offset 0  
52         // MSTORE            ← store 42 at mem[0x00]  
6020       // PUSH1 0x20        ← length = 32 bytes  
6000       // PUSH1 0x00        ← offset = 0  
f3         // RETURN            ← return(mem[0x00:0x20])  
```

Concatenate those opcodes ⇒ runtime bytecode:

```
0x602a60005260206000f3
```



## 2. Deploying Exactly 10 Bytes

In Solidity, you can bypass all the usual dispatcher code by having the constructor return those 10 bytes directly.  
Here is a minimal Solidity contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MagicNumber {
    constructor() {
        assembly {
            // Write our 10-byte “return 42” code into memory[0x00..0x1f]
            mstore(0x00, 0x602a60005260206000f300000000000000000000000000000000000000000000000000)
            // Return exactly the last 10 bytes (starting at offset 32-10 = 22 = 0x16)
            return(0x16, 0x0a)
        }
    }
}
```

- `mstore(0x00, <32-byte word>)` places the code into memory `[0x00 .. 0x1f]`
- `return(0x16, 0x0a)` slices out exactly the lower 10 bytes:
  `602a60005260206000f3` — this becomes the runtime code



## 3. Step-by-Step in Remix (or Hardhat)

1. **Copy & paste** the `MagicNumber` contract above into `MagicNumber.sol` and compile with `pragma solidity ^0.8.0;`

2. In **Remix → Deploy & Run Transactions**:
   - **Environment** → Injected Web3 (MetaMask on the same network as Ethernaut)
   - **Value** → 0 (no ETH)
   - **Contract** → select `MagicNumber`
   - **Click Deploy** and confirm in MetaMask

→ Once deployed, the contract’s runtime bytecode is exactly:

```
0x602a60005260206000f3
```



## 4. Verify it Returns 42

In Remix’s console, create a minimal ABI and call it:

```js
const abi = [{
  "inputs": [],
  "name": "whatIsTheMeaningOfLife",
  "outputs": [{"internalType":"uint256","name":"","type":"uint256"}],
  "stateMutability": "pure",
  "type": "function"
}];

const magic = new web3.eth.Contract(abi, "<DEPLOYED_ADDRESS>");
const result = await magic.methods.whatIsTheMeaningOfLife().call();
console.log(result); // "42"
```

Even though no dispatcher is present, any call (selector or empty data) executes our 10-byte code and returns 42.



## 5. Submit

Submit the instance in Ethernaut.  
The factory will check that:

- `extcodesize(addr) == 10`
- and that calling `whatIsTheMeaningOfLife()` returns 42



## Lessons Learned

- If your constructor directly includes just the instructions you need, you don’t have to include all the extra code that figures out which function to run.
- Using `return(...)` in assembly during construction directly sets exactly what code is stored, replacing everything else.
- You don’t need function selectors if your runtime always gives the same 32-byte result. Even with no call data, the EVM runs those 10 bytes and returns 42.
- Knowing EVM opcodes helps you build tiny contracts to save gas or meet special size limits.


