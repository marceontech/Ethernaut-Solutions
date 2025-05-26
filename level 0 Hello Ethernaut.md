
# Level 0: Hello Ethernaut

The first challenge is an introduction to the Ethernaut platform. Players are tasked with simply interacting with the Ethernaut smart contract to register their presence.

## Goal

Call the `info()` function of the contract to retrieve a welcome message.

## Solution

To solve this challenge, the user must:

1. Deploy the contract and connect to it using a Web3 provider (like MetaMask).
2. Call the `info()` function through the console or an interface to read the welcome message.

###  Example in JavaScript (Web3)

```
const instance = await Ethernaut.at(contractAddress);
const message = await instance.info();
console.log(message); // Displays "Hello Ethernaut!" 

