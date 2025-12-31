# ICP for EVM Developers

This course helps engineers with an Ethereum background quickly grasp smart contract development on the Internet Computer Protocol (ICP). Our goal is to streamline your learning journey towards proficiently utilizing ICP’s **Chain Fusion Technology**.

### Chain Fusion Technology

ICP smart contracts are able to make transactions to EVM-compatible chains _(e.g. Ethereum, Optimism, and Base)_, as well as non-EVM chains like Solana and Bitcoin. This feature of the ICP blockchain is called **Chain Fusion Technology.**

<figure><img src=".gitbook/assets/chain fusion technology.jpg" alt=""><figcaption></figcaption></figure>

Chain Fusion Technology relies on two key features of the ICP Protocol: **HTTP Outcalls** and **Threshold Signatures**. Briefly,

* **HTTP Outcalls** allow ICP smart contracts to perform native HTTP requests to external APIs.
* **Threshold Signatures** enable canisters on the Internet Computer to generate their own unique ECDSA and Schnorr signatures directly on-chain to sign arbitrary messages or transactions.

Combined, these features enable canisters to create signed artifacts or transactions and submit them directly to Ethereum RPCs and Solana RPCs.

This course focuses on Ethereum and Solana’s integration with ICP smart contracts, with future modules potentially covering the Internet Computer’s [direct Bitcoin integration](https://internetcomputer.org/docs/build-on-btc/).

### Hands-On Learning Journey

This course focuses on practical learning and efficiency. Where possible, we’ll put your existing Solidity experience to use. Bite-sized exercises are included to solidify ICP concepts, and key architectural ideas are explained through supporting articles.

#### Projects You’ll Build

* DeFi vaults
* Co-processors for Ethereum (Outsource heavy computations to canisters)
* Web3 oracles
* Cross-chain messaging protocols

Towards the end of this course, you’ll build a cross-chain bridge canister that facilitates message passing and token bridging between EVM chains (EVM ↔ EVM), as well as between EVM and Solana (EVM ↔ Solana).

#### Rust Prerequisites

This course assumes you already know Rust, including ownership, borrowing, and basic datatypes. Since these topics have been extensively covered, we will focus on programming canisters in Rust.
