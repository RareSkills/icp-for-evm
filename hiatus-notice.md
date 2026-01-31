# Hiatus Notice

Congratulations on making it through all 20 chapters of this course! You now have a strong foundation in building canisters in Rust, working with system APIs, and managing state-changes of atomic and asynchronous calls. That’s a big milestone — and a great place to pause.

While this course is taking a short break, you are well-equipped to keep exploring on your own. To support you during this hiatus, we’ve put together a self-guided roadmap with carefully selected external resources. These will help you continue learning further more about Chain Fusion Technology at your own pace.

## HTTP Outcalls, Threshold Signatures and ICP’s Blockchain Architecture

### **Http Outcalls**

Overview:

- https://docs.internetcomputer.org/building-apps/network-features/using-http/https-outcalls/overview

Make GET requests

- https://docs.internetcomputer.org/building-apps/network-features/using-http/https-outcalls/post
- https://docs.internetcomputer.org/building-apps/network-features/using-http/https-outcalls/get

Make POST requests:

- https://docs.internetcomputer.org/building-apps/network-features/using-http/https-outcalls/post
- https://github.com/dfinity/examples/tree/master/rust/send_http_post

### **ICP Blockchain Architecture**

Subnet architecture (independent replicated networks)

- https://docs.internetcomputer.org/building-apps/essentials/network-overview

State and transactions are private by default 

- https://docs.internetcomputer.org/building-apps/security/misc#data-confidentiality-on-icp

Each subnet has a subnet public key (used to verify its signatures/certificates)

- https://learn.internetcomputer.org/hc/en-us/articles/34209540682644-Subnet-Keys-and-Subnet-Signatures

Consensus uses threshold cryptography (Block history is no longer needed, we can verify blocks through each subnet’s public keys)

- https://learn.internetcomputer.org/hc/en-us/articles/34207558615956-Consensus

### **Threshold Signatures ECDSA**

Overview:

- https://docs.internetcomputer.org/building-apps/network-features/signatures/t-ecdsa

Derive an address (key id + derivation path)

- https://docs.internetcomputer.org/building-apps/network-features/signatures/t-ecdsa#obtaining-public-keys

Create signed artifacts

- https://docs.internetcomputer.org/building-apps/network-features/signatures/t-ecdsa#signing-messages-and-transactions

Verify signature of the signed artifact 

- https://docs.internetcomputer.org/building-apps/chain-fusion/ethereum/using-eth/signing-transactions#build-a-transaction

## The EVM RPC Canister and Timers

### EVM RPC canister:

Query Ethereum Events and Transactions

- https://docs.internetcomputer.org/building-apps/chain-fusion/ethereum/evm-rpc/evm-rpc-canister#get-the-latest-ethereum-block-info

How to send Ethereum Transactions

- https://docs.internetcomputer.org/building-apps/chain-fusion/ethereum/evm-rpc/evm-rpc-canister#get-the-latest-ethereum-block-info

### Timers

- https://docs.internetcomputer.org/building-apps/network-features/periodic-tasks-timers#timers
- https://github.com/AymericRT/On-ChainOracle

## Cross-Chain Projects

### `Project:` Build a financial subscription

- Canisters as a financial subscription service
- Credits user balance through a token approval from the owner to the canister
- https://github.com/AymericRT/SubscriptioinTimerICP

### `Project` : Cross-Chain Messaging Protocol

- Utilize canisters to relay messages from a smart contract on Chain A to a smart contract on Chain B
- https://github.com/AymericRT/ICBridge?tab=readme-ov-file