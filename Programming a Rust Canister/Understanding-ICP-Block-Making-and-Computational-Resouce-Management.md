# Understanding ICP Block Making and Computational Resource management

In Ethereum, we are used to the understanding that transactions, state changes and the entire history of the blockchain is transparent and visible through the lens of a global block explorer such as [Etherscan.io](http://etherscan.io).

However, we cannot exactly carry over this understanding as we learn to develop on the IC. The Internet Computer Protocol does not have a global block explorer in the same sense as Ethereum’s block explorer.

## The Absence of a Global Block Explorer on the Internet Computer

The Internet Computer Protocol introduces a level of privacy that makes the transaction data within finalized blocks not publicly accessible information. Therefore, it rests the responsibility of producing the block history and the audit trail of the smart to the developer.

There are only a few components that are publicly accessible in the ICP Network.

* The subnet of the contract.
* The Canister id. ( The contract address )
* The hash of the [WASM](https://webassembly.org/) module of the contract (This is equivalent to the Bytecode in Solidity )
* The controllers of the contract. ( This is almost equivalent to the deployers of the contract )

### Block making

Blocks extending the ICP network are regularly pruned. This means that unlike Ethereum, which the history from the genesis block is stored by every node validators, ICP does not store that data among its validators. This design is by choice as ICP focuses on maintaining the current global state of ICP so that the network can operate more efficiently and offer developers more control over the transparency of their applications.

It would almost be infeasible to track transactions across the network with this architecture. Hence, the state and block history of the Canister, becomes the responsibility of the Canister Developer to manage and build it.

### Subnets

The Internet Computer (IC) is architecturally divided into subnets, with each subnet functioning as its own blockchain. These subnets operate independently but can communicate with each other through a message routing protocol, enabling cross-subnet interactions and the cohesive functioning of the entire network. At the heart of the Internet Computer's architecture is the [**Network Nervous System (NNS)**](https://internetcomputer.org/nns) that is responsible for overseeing and governing the entire ICP network.

## Each canister needs to manage its own blocks and transactions

Due to the absence of a global block explorer, developers must implement their own mechanisms for tracking state changes and transactions within their canisters in order to have a functional audit trail. For example, the ICP Ledger tracks all transactions involving ICP tokens. The [ICP dashboard](https://dashboard.internetcomputer.org) provides an interface to view these transactions, showcasing how a canister can manage and display its own transaction history.

![ICP Block Making](../.gitbook/assets/ICPBlock.png)

The block that we verified in the previous article was implemented at the canister level, not the ICP protocol level. This is what we meant by it is the developer’s duty to track the state changes and history.

### Accounts in ICP does not have native balances in the same way as Ethereum

Accounts in ICP does not have a built in balance tracker. All accounts in Ethereum have a native balance to track ether. This concept does not carry over to Internet Computer.

If ICP Tokens are implemented in as a canister, then what is used as the computational resource currency? It is the cycles. Cycles are obtained by converting ICP tokens to cycles. You can think of cycles as the currency used for using the network.

The Cycles Wallet keeps track of cycles

| Smart Contract      | Canister                         |
| ------------------- | -------------------------------- |
| Address             | Canister ID                      |
| Bytecode            | Module Hash                      |
| Storage             | Memory (opaque)                  |
| Ether Balance       | —                                |
| Nonce               | —                                |
| Transaction History | Self managed                     |
| Deployer            | Canister Controller              |
| Compiler version    | Wasm Compiler Verision (Public?) |

## The computation currency of the Internet Computer is not the ICP Token, but `cycles`.

In Ethereum, storing data and submitting transactions to the network uses up ether as its gas fee.

Gas fees in Ethereum serve as a payment mechanism for computational resources used on the network. They can be highly variable and unpredictable, often increasing dramatically during periods of high network congestion.

In contrast, the Internet Computer uses cycles for a similar purpose, but with some key differences:

* Cycles are convertible from ICP. The value of cycles is pegged to the SDR (Special Drawing Rights, an international reserve asset). 1 trillion cycles is always almost equal to 1 SDR. This allows computation to be predictable and stable.

The Cycles Ledger on the Internet Computer manages these cycles. They are denominated in thousands, millions, and billions, providing a flexible range of units for different computational needs. This system aims to provide more stability and predictability in costs for developers and users compared to Ethereum's gas fee model. The pegging to SDR helps insulate the cost of computation from the volatility often seen in cryptocurrency markets. For more detailed information, you can refer to:

* https://internetcomputer.org/docs/current/developer-docs/defi/cycles/cycles-ledger
* https://dashboard.internetcomputer.org/canister/um5iw-rqaaa-aaaaq-qaaba-cai
