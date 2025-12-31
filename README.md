# ICP for EVM Developers

This course helps engineers with an Ethereum background quickly grasp smart contract development on the Internet Computer Protocol (ICP). Our goal is to streamline your journey towards learning how to program ICP smart contracts and utilize ICP’s **Chain Fusion Technology**.

### Chain Fusion Technology

ICP smart contracts have the ability to create and send transactions to EVM-compatible blockchains _(e.g. Ethereum, Optimism, and Base)_, as well as non-EVM chains like Solana or Bitcoin. This feature of the ICP blockchain is called **Chain Fusion Technology.**

Chain Fusion Technology is made possible by two key features of the Internet Computer Protocol: **HTTP Outcalls** and **Threshold Signatures**. Briefly,

* **HTTP Outcalls** allow ICP smart contracts to perform native HTTP requests to external APIs.
* **Threshold Signatures** enable canisters on the Internet Computer to generate their own unique ECDSA and Schnorr signatures directly on-chain to sign arbitrary messages or transactions.

Combined, these features enable canisters to create signed artifacts or transactions and submit them directly to Ethereum RPCs and Solana RPCs.

We will focuses on learning how to program an ICP canister written in Rust as well as its integration with the Ethereum and Solana blockchain. Future modules may potentially cover ICP's [direct Bitcoin integration](https://internetcomputer.org/docs/build-on-btc/).

### A Hands-On Learning Journey

This course focuses on practical learning and efficiency. Where possible, we’ll put your existing Solidity experience to use. Bite-sized exercises are included to solidify ICP concepts, and key architectural ideas are explained through supporting articles.

#### Projects You’ll Build

* DeFi vaults
* Co-processors for Ethereum (Outsource heavy computations to canisters)
* Web3 oracles
* Cross-chain messaging protocols

Towards the end of this course, you’ll build a cross-chain bridge canister that facilitates message passing and token bridging between EVM chains (EVM ↔ EVM), as well as between EVM and Solana (EVM ↔ Solana).

#### Rust Prerequisites

This course assumes you already know Rust, including ownership, borrowing, and basic datatypes as these topics have been extensively covered by outside resources.
