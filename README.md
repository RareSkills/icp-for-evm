# ICP for EVM Developers

This course helps engineers with an Ethereum background quickly grasp smart contract development on the Internet Computer Protocol (ICP). Our goal is to streamline your journey towards learning how to program ICP smart contracts and utilize ICP’s [**Chain Fusion Technology**](https://learn.internetcomputer.org/hc/en-us/articles/34329023770260-Chain-Fusion).

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

* Fungible tokens
* Web3 oracles
* Co-processors for Ethereum (Outsource heavy computations to canisters)
* Cross-chain messaging protocols

Towards the end of this course, you’ll build a cross-chain bridge canister that facilitates message passing and token bridging between EVM chains (EVM ↔ EVM), as well as between EVM and Solana (EVM ↔ Solana).

### Rust Prerequisites

This course assumes you already know Rust, including ownership, borrowing, and basic datatypes as these topics have been extensively covered by outside resources.

### Acknowledgement

We would like to thank Ujjwal, [Faybian Byrd](https://www.linkedin.com/in/faybianbyrd/), Serah and Moritz for their careful reviews and meaningful contributions. This work was supported by a grant from the Dfinity foundation.

### Course Outline

#### Module 1: Programming a Rust Canister

{% content-ref url="Module 1: Programming a Rust Canister/Chapter 1 Hello World and Developer Environment Installation.md" %}
[Chapter 1 Hello World and Developer Environment Installation.md](<Module 1: Programming a Rust Canister/Chapter 1 Hello World and Developer Environment Installation.md>)
{% endcontent-ref %}

{% content-ref url="Module 1: Programming a Rust Canister/Chapter 2  Function Visibility and Mutability.md" %}
[Chapter 2  Function Visibility and Mutability.md](<Module 1: Programming a Rust Canister/Chapter 2  Function Visibility and Mutability.md>)
{% endcontent-ref %}

{% content-ref url="Module 1: Programming a Rust Canister/Chapter 3 State Storage Variables.md" %}
[Chapter 3 State Storage Variables.md](<Module 1: Programming a Rust Canister/Chapter 3 State Storage Variables.md>)
{% endcontent-ref %}

{% content-ref url="Module 1: Programming a Rust Canister/Chapter 4 The Update Function #[ic_cdk update].md" %}
[Chapter 4 The Update Function #\[ic\_cdk update\].md](<Module 1: Programming a Rust Canister/Chapter 4 The Update Function #\[ic_cdk update].md>)
{% endcontent-ref %}

{% content-ref url="Module 1: Programming a Rust Canister/Chapter 5 Safe Storage and Access With Thread-Local Storage.md" %}
[Chapter 5 Safe Storage and Access With Thread-Local Storage.md](<Module 1: Programming a Rust Canister/Chapter 5 Safe Storage and Access With Thread-Local Storage.md>)
{% endcontent-ref %}

{% content-ref url="Module 1: Programming a Rust Canister/Chapter 6 Composite Types.md" %}
[Chapter 6 Composite Types.md](<Module 1: Programming a Rust Canister/Chapter 6 Composite Types.md>)
{% endcontent-ref %}

#### Canister Initialization, System APIs, and Error Handling

{% content-ref url="Module 2: Canister Initialization, System APIs, and Error Handling/Chapter 7 Initialize Canister State and Canister Upgrades.md" %}
[Chapter 7 Initialize Canister State and Canister Upgrades.md](<Module 2: Canister Initialization, System APIs, and Error Handling/Chapter 7 Initialize Canister State and Canister Upgrades.md>)
{% endcontent-ref %}

{% content-ref url="Module 2: Canister Initialization, System APIs, and Error Handling/Chapter 8 System-Level Information In ICP.md" %}
[Chapter 8 System-Level Information In ICP.md](<Module 2: Canister Initialization, System APIs, and Error Handling/Chapter 8 System-Level Information In ICP.md>)
{% endcontent-ref %}

{% content-ref url="Module 2: Canister Initialization, System APIs, and Error Handling/Chapter 9 Traps and Error Handling.md" %}
[Chapter 9 Traps and Error Handling.md](<Module 2: Canister Initialization, System APIs, and Error Handling/Chapter 9 Traps and Error Handling.md>)
{% endcontent-ref %}

{% content-ref url="Module 2: Canister Initialization, System APIs, and Error Handling/Chapter 10 Simple Token Canister Project.md" %}
[Chapter 10 Simple Token Canister Project.md](<Module 2: Canister Initialization, System APIs, and Error Handling/Chapter 10 Simple Token Canister Project.md>)
{% endcontent-ref %}

#### ICP Token, System Canisters and The Reverse Gas Model

{% content-ref url="Module 3: ICP Token, System Canisters and The Reverse Gas Model/Chapter 11 Interact with Canisters Using The ICP JavaScript Agent.md" %}
[Chapter 11 Interact with Canisters Using The ICP JavaScript Agent.md](<Module 3: ICP Token, System Canisters and The Reverse Gas Model/Chapter 11 Interact with Canisters Using The ICP JavaScript Agent.md>)
{% endcontent-ref %}

{% content-ref url="Module 3: ICP Token, System Canisters and The Reverse Gas Model/Chapter 12 System Canisters (NNS) and The ICP Token.md" %}
[Chapter 12 System Canisters (NNS) and The ICP Token.md](<Module 3: ICP Token, System Canisters and The Reverse Gas Model/Chapter 12 System Canisters (NNS) and The ICP Token.md>)
{% endcontent-ref %}

{% content-ref url="Module 3: ICP Token, System Canisters and The Reverse Gas Model/Chapter 13 The Reverse Gas Model of ICP.md" %}
[Chapter 13 The Reverse Gas Model of ICP.md](<Module 3: ICP Token, System Canisters and The Reverse Gas Model/Chapter 13 The Reverse Gas Model of ICP.md>)
{% endcontent-ref %}

{% content-ref url="Module 3: ICP Token, System Canisters and The Reverse Gas Model/Chapter 14 Canisters Pay for Storage.md" %}
[Chapter 14 Canisters Pay for Storage.md](<Module 3: ICP Token, System Canisters and The Reverse Gas Model/Chapter 14 Canisters Pay for Storage.md>)
{% endcontent-ref %}

#### Inter-Canister Communication and Asynchronous Execution

{% content-ref url="Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 15 Inter-Canister Calls.md" %}
[Chapter 15 Inter-Canister Calls.md](<Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 15 Inter-Canister Calls.md>)
{% endcontent-ref %}

{% content-ref url="Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 16 Decoding and Encoding Inter-Canister Arguments.md" %}
[Chapter 16 Decoding and Encoding Inter-Canister Arguments.md](<Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 16 Decoding and Encoding Inter-Canister Arguments.md>)
{% endcontent-ref %}

{% content-ref url="Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 17 unbounded_wait() vs bounded_wait().md" %}
[Chapter 17 unbounded\_wait() vs bounded\_wait().md](<Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 17 unbounded_wait() vs bounded_wait().md>)
{% endcontent-ref %}

{% content-ref url="Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 18 Asynchronous Execution And It’s Effect On State-Changes.md" %}
[Chapter 18 Asynchronous Execution And It’s Effect On State-Changes.md](<Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 18 Asynchronous Execution And It’s Effect On State-Changes.md>)
{% endcontent-ref %}

{% content-ref url="Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 19 Canisters are Non-Blocking.md" %}
[Chapter 19 Canisters are Non-Blocking.md](<Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 19 Canisters are Non-Blocking.md>)
{% endcontent-ref %}

{% content-ref url="Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 20 Transfer Cycles Between Canisters.md" %}
[Chapter 20 Transfer Cycles Between Canisters.md](<Module 4: Inter-Canister Communication and Asynchronous Execution /Chapter 20 Transfer Cycles Between Canisters.md>)
{% endcontent-ref %}

#### Module 5: Coming Soon

### Proposing changes

If you want to suggest an edit or improvement, please open an **Issue** or submit a **Pull Request** on the GitHub repo: [https://github.com/RareSkills/icp-for-evm](https://github.com/RareSkills/icp-for-evm).

### Additional Resources

* **Community Forum**: [ICP Developer Forum](https://forum.dfinity.org/)
* **Discord**: [Developer Discord](https://discord.internetcomputer.org/)
* **YouTube Tutorials**: [DFINITY YouTube Channel](https://www.youtube.com/c/DFINITY)
* **Developer Grants**: [ICP Community Developer Grants](https://dfinity.org/grants)
