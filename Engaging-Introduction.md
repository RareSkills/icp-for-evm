# ICP for EVM Developers

This tutorial is a fast-paced, hands-on guide designed for engineers with an Ethereum development background to quickly grasp development on ICP (**I**nternet **C**omputer **P**rotocol). Through leveraging your existing knowledge of the Ethereum blockchain, you do not have to start from scratch.

Our goal is to fast track your learning journey to effectively use ICP’s [**Chain Fusion Technology**](https://internetcomputer.org/chainfusion), an innovative approach that allows ICP smart contracts or in IC nomenclature, “Canisters”, to seamlessly communicate with other blockchain protocols such as Ethereum, Bitcoin, and soon, Solana. 

### ICP’s Chain Fusion Technology (CFT)

The Chain Fusion Technology feature allows Canisters (ICP smart contracts) to interact with contracts on Ethereum, EVM-compatible chains, and also the Bitcoin network. This allows ICP smart contracts to communicate securely between multiple blockchains.

Leveraging this feature, we can build a variety of functionalities, including co-processors, bridges, and ICP smart contracts capable of holding multiple tokens across various blockchains, enabling the creation of cross-chain DeFi applications, multi-chain marketplaces, cross-chain governance systems, and numerous other innovative solutions.

Some popular CFT implementations are of ckBitcoin, ckETH and ckUSDC, these are the digital twins of BTC, ETH and USDC on the ICP network. One advantage is that you can trade the digital twin of Bitcoin, Eth cheaply. 

Soon, Chain Fusion Technology can be used to communicate with the Solana network after their next [Helium](https://internetcomputer.org/roadmap#Chain%20Fusion-Helium) milestone.

## Both the Internet Computer Protocol (ICP) and Ethereum are fundamentally “Blockchains”

Both the Internet Computer (IC) and Ethereum are fundamentally “Blockchains.” Despite their different approaches to running the blockchain, they share the fundamental characteristics of decentralized state machines that maintain and update a global shared state. They employ consensus mechanisms to ensure the integrity and security of the blockchain. 

While their architectures and specific implementations may vary, both platforms aim to provide a secure, decentralized environment for executing smart contracts.

## We begin by exploring the similarities of ICP and Ethereum

Instead of diving straight into all the differences, this tutorial focuses on distilling essential information through this approach:

“If I know how to do X in Ethereum, how do I accomplish X in ICP?”

And in some situations,

“Why can’t I do X in ICP?”

We choose this method because it’s easier to understand new concepts if you can relate them to something you’re already familiar with.

Our objective is to help you grasp both the similarities and key differences between ICP and Ethereum.

(the principles discussed also apply to other EVM-compatible blockchains like ZKsync and Base.)

## Which programming language should I use?

The Internet Computer supports several languages for developing smart contracts, including but not limited to Motoko (ICP’s native programming language),Rust, TypeScript, Python, and Solidity. 

For the purpose of this tutorial, we will provide our code examples in Rust and Motoko, as these languages are maturely developed and production ready.

## Internet Computer Course

### Module 1

- **Native Token Transfer and Verification in ICP** 
- **Understanding ICP Block Making and Computational Resource management**
- Entry point nodes in Ethereum and ICP

### Module 2

- Deploy a canister locally and show how to interact with it
- Write a smart contract that on a call, makes a native token transaction (ICP).
- Project: Create a simple Bank Canister on ICP

### Module 3

- Asynchronous Execution on ICP