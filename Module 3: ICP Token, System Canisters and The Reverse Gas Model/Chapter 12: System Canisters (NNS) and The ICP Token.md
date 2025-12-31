# System Canisters (NNS) and The ICP Token

In this tutorial, we’ll simulate `ICP` tokens on our local IC replica and transfer them using both the `dfx` CLI and the ICP JavaScript Agent.

The network’s native token, `ICP`, is implemented similarly to an ERC-20 token. Balances and transfers are managed by a system canister called the **ICP Ledger**. This ledger is part of the Network Nervous System (NNS), a set of core system canisters built into the protocol.

On both local the IC instance and mainnet, the ICP Ledger has the same canister ID: `ryjl3-tyaaa-aaaaa-aaaba-cai`.

By default, a local replica doesn’t start with the NNS. So to test `ICP` token transfers locally, we need to start dfx with the NNS enabled.

## Set up the NNS Locally

To initialize our local IC instance with the NNS system canisters, add the `—system-canister` flag to our `dfx start` command. 

```bash
 dfx start --system-canisters --clean --background
```

- The `--clean` flag clears any existing local state before starting (fresh local replica).
- The `--background` flag runs the local replica in the background so your terminal is free for other commands.

Within the localnet, the anonymous principal is automatically minted with **1 Billion** ICP Tokens. (At time of writing 1 ICP = $3.23. So 1 billion ICP equals 3.23 billion dollars).

On the mainnet however, you would need to initially get `ICP` from a CEX or a friend.

## The Anonymous Principal is minted with 1 Billion ICP Tokens

To start sending ICP tokens out, we’ll first need to tell dfx to use the anonymous identity.

### Switch dfx Identities to the anonymous principal

The command below tells dfx to use anonymous identity. Which just means that the transaction is unsigned (no public/private key pair).

```jsx
dfx identity use anonymous
```

The `dfx identity use <identity_name>` command is used to alter between identities saved in dfx. You can see the list of identities that dfx saves using the `dfx identity list` command.

Confirm that you are using the anonymous principal. Unsigned transactions are identified by the principal:  `2vxsx-fae`.

```jsx
dfx identity get-principal
```

![Screenshot 2025-10-22 at 04.51.04.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/Screenshot_2025-10-22_at_04.51.04.png)

### Check anonymous’ ICP Balance

To check the balance of the `anonymous` identity, run

```bash
dfx ledger balance
```

![Screenshot 2025-07-28 at 01.22.15.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/Screenshot_2025-07-28_at_01.22.15.png)

## ICP Ledger `Transfer()` function

To transfer ICP Tokens, we would be calling the `transfer()` function of the ICP Ledger Canister. It takes the following required arguments:

- `To`: Account Identifier
- `Amount`:  ICP amounts denoted by `e8s`. `100_000_000 e8s` = 1 ICP Token
- `Fee`: 10_000 `e8s` or 0.0001 ICP.
- `Memo`: Arbitrary u64 number.

ICP transfers are directed to **account identifiers**, not principals.

### Transfers are directed to `Account Identifiers`

To save memory, the ICP Ledger hashes Principals into a **32-byte hash** called the `account identifier`, reducing storage requirements by ~50%.

The general formula for computing the account_id from the principal is

```
accountId = hash(principal, subaccount)
```

Here, the `subaccount` acts like a salt. By changing the `subaccount` value, you derive different account IDs that are all tied to the same principal that is still under your control

```
accountId1 = hash(principal, null)
accountId2 = hash(principal, 2)
```

We can query the default account identifier for genesis (`null` value for subaccount) using dfx

```bash
dfx ledger account-id
```

![Screenshot 2025-10-22 at 04.52.57.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/Screenshot_2025-10-22_at_04.52.57.png)

**anonymous** would have the default account identifier of: `1c7a48ba6a562aa9eaa2481a9049cdf0433b9738c992d698c31d8abf89cadc79`

Optionally, you could also compute for the account_id of a principal through calling the [`account_identifier()` function](https://dashboard.internetcomputer.org/canister/ryjl3-tyaaa-aaaaa-aaaba-cai) on NNS ICP Ledger dashboard. `dfx` abstracts this function call for you.

![Screenshot 2025-08-07 at 19.12.20.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/Screenshot_2025-08-07_at_19.12.20.png)

## Transfer `ICP` through dfx

We’ll transfer 5 ICP from anonymous to a random identity. Let’s call him `Bob`.

`Bob` has a user **principal** of:

- `ncgnw-oidst-l5kdp-3go53-cegtn-aql23-ync6p-himc7-gm6w7-3s2tw-xae`.

His default **account-id** would be:

- `5dbfde09991c7a6d3f5208df7328e7ea54109ee3b63fcc4bc1e44de020ec2b8a`.

The default **Fee** is fixed at **10_000** `e8s` and we’ll set the **memo** to **12345**.

In summary the transfer to Bob would have the following parameters:

- `To` : `ncgnw-oidst-l5kdp-3go53-cegtn-aql23-ync6p-himc7-gm6w7-3s2tw-xae`
- `Amount` : `500_000_000`
- `Fee` : `10_000`
- `memo` : `12345`

Using dfx’s built in ledger transfer function,

```bash
dfx ledger transfer <recipient's account identifer> --icp <ICP amount> --memo <arbitrary number>
```

Replace the details from above.

```bash
dfx ledger transfer be3d302a9fcb6da93418f33bfc85b6d58c6b11d4670df736326257d13321191a --e8s 500000000 --memo 12345
```

A successful output should indicate the block which your transaction proof is stored at.

![Screenshot 2025-08-07 at 19.35.29.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/Screenshot_2025-08-07_at_19.35.29.png)

Your first transaction should be at block **5**.

## Transfer ICP using ICP Javascript Agent

Install the necessary dependencies to use the `Agent JS`.  Since the anonymous account is an unsigned transaction, we do not have to import a private key.

1. Create a new folder called `AgentJs`. In that folder directory, run the following commands: 
    
    ```rust
    npm init -y
    npm i --save @dfinity/agent
    npm i @dfinity/identity-secp256k1 
    ```
    
    Your folder directory should look like this:
    
    ![Screenshot 2024-08-29 at 14.35.36.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/Screenshot_2024-08-29_at_14.35.36.png)
    
2. Create a new file called `idlFactory.js` and copy paste the **Javascript** version of the Candid Interface. This is similar to creating a separate file for your contract ABI in Ethereum. 

The **Javascript** **Candid interface** content can be found at the bottom of the ICP Ledger Page page [here](https://dashboard.internetcomputer.org/canister/ryjl3-tyaaa-aaaaa-aaaba-cai). 
    
    ![Screenshot 2024-10-02 at 14.00.27.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/ab1caa47-52ed-4f64-87d1-b7cc07263c05.png)
    
    Your folder should look similar to this:
    
    ![Screenshot 2024-08-29 at 14.48.24.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/d78da356-2a87-4e40-b221-fb7b04b37358.png)
    
3. Within the 📁AgentJS folder, create a new file called `agentScript.js` and import the modules:
    
    ```jsx
    import { Secp256k1KeyIdentity } from '@dfinity/identity-secp256k1';
    import { HttpAgent, Actor } from "@dfinity/agent";
    import { Principal } from "@dfinity/principal";
    import { idlFactory } from "./idlFactory.js";
    import fs from 'fs';
    ```
    
    ![Screenshot 2024-08-29 at 15.00.39.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/Screenshot_2024-08-29_at_15.00.39.png)
    

From now on, the following codes will be added to `agentScript.js` 

1. Create an **agent** instance. (This is similar to creating a “wallet” instance in Ethers.js.)
    
    ```jsx
    // Imported Libraries...
    
    // Create agent
    const agent = await HttpAgent.create({
      host: "http://127.0.0.1:8080", // connect to the local host
      fetch,
    });
    await agent.fetchRootKey();
    console.log("🌐 Agent ready.");
    ```
    
2. Declare the ICP Ledger Canister instance, or in ICP terms, `Actor`. Canister instances are referred to as `Actors` in ICP:
    
    ```jsx
    // previous code...
    
    // Ledger canister ID and actor creation
    const ledgerCanisterId = "ryjl3-tyaaa-aaaaa-aaaba-cai";
    const actor = Actor.createActor(idlFactory, {
      agent,
      canisterId: ledgerCanisterId,
    });
    console.log("📦 Actor created.");
    ```
    

1. **Build** the transaction data:
    
    ```jsx
    // previous code...
    
    // Target Receiver Principal
    const receiverPrincipalStr = "jvjbz-wuejr-a6thn-gkzt4-ybkfq-slmbo-2w3ku-kgnjf-pg6x7-xxfqs-vae";
    const ReceiverPrincipal = Principal.fromText(receiverPrincipalStr);
    const account = { owner: ReceiverPrincipal, subaccount: [] };
    
    const accountId = await actor.account_identifier(account);
    console.log("🔑 Resolved accountId:", accountId);
    
    // Prepare transfer
    const TransferArgs = {
      to: accountId,
      fee: { e8s: BigInt(10_000) },
      memo: BigInt(0),
      from_subaccount: [],
      created_at_time: [],
      amount: { e8s: BigInt(1_00_000_000) },
    };
    ```
    
2. **Sign** and **Send** the transaction:
    
    ```jsx
    // previous code ...
    try {
        const result = await actor.transfer(TransferArgs);
        console.log("Transfer result:", result);
    } catch (error) {
        console.error("Error :", error);
    }
    ```
    

Upon success, you should be receiving a Result type, “Ok: 15n” along with the “transaction block”:

![Screenshot 2025-07-21 at 13.26.42.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/Screenshot_2025-07-21_at_13.26.42.png)

The return value of a successful transfer is the index of the block that includes the transaction. Let’s verify it.

## Verify the ICP Token Transaction

Successful ICP transactions are recorded by the ledger canister. In the previous example, the `dfx ledger transfer`command returned `Ok:15n`, which means the transaction was recorded in block index 15.

You can verify it by querying that block using dfx

```bash
dfx canister call ryjl3-tyaaa-aaaaa-aaaba-cai query_blocks '(record { start = 15 : nat64; length = 1 : nat64 })'
```

![Screenshot 2025-08-07 at 19.47.49.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/Screenshot_2025-08-07_at_19.47.49.png)

or ICP Javascript Agent

```rust
const blockHeight = BigInt(5);
const block = await actor.block(blockHeight);
console.log("Block at height 5:", block);
```

![Screenshot 2025-08-07 at 19.45.58.png](System%20Canisters%20(NNS)%20and%20The%20ICP%20Token/Screenshot_2025-08-07_at_19.45.58.png)

Your transaction details are recorded at the canister, including timestamp, sender, recipient, amount, memo, and fees.

The result confirms:

- Our unique memo: `12345`
- The Recipient: `5dbfde09991c7a6d3f5208df7328e7ea54109ee3b63fcc4bc1e44de020ec2b8a`
- The amount sent: `500_000_000 e8s` = `5 ICP`
- The memo: `12345`
- The block index: `5`

This matches the earlier command:

```bash
dfx ledger transfer ae6e1a76da5725bbbf0c5c035aaf0525b791e0f0f7cce27d8e27826389871406 --icp 5 --memo 12345
```

Note: The blocks that are queried are information that is stored on the canister-level since the protocol doesn’t have an Solidity event-like functionality.

`ICP` tokens are used for protocol-level governance voting, store of value and rewarding validators. However, they are not used for paying gas fees, another native token, `Cycles`, is used instead. The next article discusses the gas model and `Cycles` token of the Internet Computer Protocol.