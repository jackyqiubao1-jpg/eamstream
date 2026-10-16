---
title: "The Latest Ethereum Solidity Smart Contract Development Tutorial (Part -4) — Build Frontend html"
date: 
tags: [ethereum, solidity, blockchain, javascript]
draft: false
---


In previous serial articles, you learned how to setup development environment, built a solidity smart contract in truffle, and deployed it to a testing blockchain network ganache. You can refer to the following links for the previous sessions:

[(Part -1) — Solidity Smart Contract development environment setup for Visual Studio Code with Truffle framework](https://medium.com/@jackyqiubao/the-latest-ethereum-solidity-smart-contract-development-tutorial-part-1-develop-solidity-4ee15bdb1dee)

[(Part -2) — Develop solidity smart contract in truffle](https://medium.com/@jackyqiubao/the-latest-ethereum-solidity-smart-contract-development-tutorial-part-2-develop-solidity-871f743ad113)

[(Part -3) — Compile, Deploy and Test solidity smart contract in Truffle](https://medium.com/@jackyqiubao/the-latest-ethereum-solidity-smart-contract-development-tutorial-part-3-compile-deploy-and-de653ff1367f)

Now it is time for us to learn how to interact with our solidity smart contract in javascript through Web3.js.

Before we get into the detail, we might need to understand couple concepts:

**ABI:** The ABI, Application Binary Interface, is **basically how you call functions in a contract and get data back**.

> An ABI determines such details as how functions are called and in which binary format information should be passed from one program component to the next…

An Ethereum smart contract is bytecode deployed on the Ethereum blockchain. There could be several functions in a contract. An ABI is necessary so that you can specify which function in the contract to invoke, as well as get a guarantee that the function will return data in the format you are expecting.

From [**Ethereum’s ABI specification**](https://github.com/ethereum/wiki/wiki/Ethereum-Contract-ABI), an example:

```
contract Foo {
  function bar(real[2] xy) {}
  function baz(uint32 x, bool y) returns (bool r) { r = x > 32 || y; }
  function sam(bytes name, bool z, uint[] data) {}
}
```

If we wanted to call baz with the parameters 69 and true, we would pass 68 bytes in total, which can be broken down into:

> 0xcdcd77c0: the Method ID. This is derived as the first 4 bytes of the Keccak-256 hash of the ASCII form of the signature baz(uint32,bool). 0x0000000000000000000000000000000000000000000000000000000000000045: the first parameter, a uint32 value 69 padded to 32 bytes 0x0000000000000000000000000000000000000000000000000000000000000001: the second parameter — boolean true, padded to 32 bytes

The 68 bytes is what would be specified in the `data` field of a transaction: a security note on that is [here](https://ethereum.stackexchange.com/questions/29717/is-sending-some-datas-in-the-data-field-safe). (To summarise, be careful what you put in the data field, because it can have unintended, possibly malicious side-effects when passing it to the calling contract.)

To avoid a common pitfall **when deriving the Method ID, the canonical types must be used, for example **`uint256`** instead of **`uint`.

**Web3.js:** The web3.js library is a collection of modules which contain specific functionality for the Ethereum ecosystem. Using a higher-level library such as web3.js abstracts most of these details, but the ABI in JSON format still needs to be provided to web3.js. I will show you how to get ABI in JSON format later for the smart contract Rockscissorspaper we built before.
*Note: the ABI is an abstraction that is not part of the core Ethereum protocol. Anyone can define their own ABI for their contracts, and any callers of such contracts would have to comply with that ABI to get meaningful results. However, it is simpler for all developers to use Solidity, Serpent, and web3.js which all comply with the ABI above.*

Now let’s start to build html and javascript files to call Rockscissorspaper smart contract to play rock, scissors and paper game.

We gotten to build the following front end files in a new folder web:
