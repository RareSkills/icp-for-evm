# ICP Token Transfer and Verification

In this tutorial, we’ll learn how to send ICP Tokens from one account to another and verify the finalized transaction. All using `Agent.js`, the equivalent of `Ethers.js` in Ethereum.

Say Alice wants to send `1 Ether` to Bob, how will it translate to sending `1 ICP`  to Bob on the Internet Computer (IC)?

In both blockchains, Alice has to first **construct her** **transaction data**. A generalized template can be understood as:

- `Transfer | From | To | Value`, or
- `<Action>` | `<Sender Account>` | `< Receiving Account >` | `<Amount of Tokens>`

Populating the data for Alice and Bob, we get:

- `Transfer | Alice | Bob | 1 Ether` , in Ethereum and
- `Transfer | Alice | Bob | 1 ICP`, in ICP

For easier comparison, we’ll organize the comparison into a table.

|  | Send 1 Ether | Send 1 ICP |
| --- | --- | --- |
| Construct Transaction data | Transfer | Alice | Bob | 1 Ether  | Transfer | Alice | Bob | 1 ICP |

Second we’ll require Alice to **sign the transaction** **data** with her private key.

|  | Send 1 Ether | Send 1 ICP |
| --- | --- | --- |
| 1. Construct transaction data | `Transfer \ Alice \ Bob \ 1 Ether`  | `Transfer \ Alice \ Bob \ 1 ICP` |
| 2. Sign Transaction Data | `Sign(Transfer \ Alice \ Bob \ 1 Ether)` | `Sign(Transfer \ Alice \ Bob \ 1 ICP)`
 |

Lastly, the **transaction will be sent** to the network through an RPC provider such as Infura in Ethereum and the Boundry node in ICP, for it to be included into the the blockchain. 

|  | Send 1 Ether | Send 1 ICP |
| --- | --- | --- |
| 1. Construct transaction data | `Transfer \ Alice \ Bob \ 1 Ether`  | `Transfer \ Alice \ Bob \ 1 ICP` |
| 2. Sign Transaction Data | `Sign(Transfer \ Alice \ Bob \ 1 Ether)` | `Sign(Transfer \ Alice \ Bob \ 1 ICP)`
 | 3. Submit signed transaction to the blockchain | Using an RPC provider like Infura, or other providers, we can submit our transaction for it to be included into the network | Using ICP’s own boundary node, the equivalent of Ethereum’s RPC Provider. |

\
The steps described above are common procedures among blockchains, including the Internet Computer. 

## Sending ICP Token using the `Agent.js` Javascript Library

Like `Ethers.js`, we can use `Agent.js` to write a script that directly interacts with the Internet Computer Protocol and transfer `1 ICP` from Alice to Bob. We'll also demonstrate how this process is similar to sending `1 Ether` on Ethereum using `Ethers.js`.

First, we’ll need to create an ICP account and obtain its address or in ICP terms, the principal. We’ll also need to know the decimal point precision of the ICP Token as well as top-up our wallet with ICP. 

Network…? Should i explain that this is in the mainnet, the readers might be discouraged to use real funds. We need some simulated funds

### ICP Nomenclature: The Principal = Address

In Ethereum, we are used to interacting with the 20 bytes address such as `0xAb5801a7D398351b8bE11C439e05C5B3259aeC9B`. This address is derived from the last 20 bytes of the keccak256 of the original public key (64 bytes).

In the Internet Computer, they have a different nomenclature. Addresses in IC are referred to as **Principals**, and as their derivation path and formatting is different, it results into a different address. The **principal,** a term ****which you would be seeing often in your ICP development journey, would look similar to `ybdt7-ozynu-jnybe-4hyp4-rna4b-s4343-mxmfd-46wqq-w72ia-2h2sr-hae`. 

| Ethereum Terms | ICP Terms |
| --- | --- |
| Smart Contract | Canister |
| Address | Principal |

### ICP Token Decimal Precision

Ether uses 18 decimal places to represent 1 Ether Token. ICP Tokens, uses 8 decimal point precision. Meaning that 1_00_000_000 = 1 ICP token.   

### A quick comparison to sending Ether using `Ethers.js`

The steps below are quite simple and straight forward. We need to obtain our private and public key pair. Next build the transaction data, sign the transaction data and submit it through the RPC. 

1. Import the `Ethers.js` Library

```jsx
const { ethers } = require('ethers');
```

1. Create a **new wallet** instance with **Private Key** and **Provider**

```jsx
// Private key and Provider
const provider = new ethers.providers.InfuraProvider('mainnet', 'YOUR_INFURA_PROJECT_ID');
const senderPrivateKey = 'your-private-key'; // Never share or hardcode your private key in production!
const wallet = new ethers.Wallet(senderPrivateKey, provider);
```

1. **Build** and **Submit** **Transaction**

```jsx
// Function to build and submit transaction
const transferEth = async () => {
  // Amount and Receiver detail
  const receiverAddress = '0xReceiverAddress';
  const amount = ethers.utils.parseEther('0.1'); // Amount to send in etherÏ
  // Build transaction data
  const tx = {
    to: receiverAddress,
    value: amount,
  };

  // Sign and Submit transaction to the Infura Provider
  const transaction = await wallet.sendTransaction(tx);

  // Proof of receipt (for verification)
  const receipt = await transaction.wait();
};

// calling the function
transferEth().catch(console.error);
```

Libraries like Ethers.js and Agent.js helps us abstract a lot of the transaction overhead. The idea is that we understand how transactions are proposed in Ethereum, and find its equivalent in the Internet Computer Protocol.

Full Github Repository: 

### The `Agent.js` Implementation

ICP token are implemented as the equivalent of an Erc20 smart contract. If Alice wants to transfer 1 ICP token to Bob. She would have to call the `transfer` function on the **ICP Ledger canister**.

The [ICP Ledger](https://dashboard.internetcomputer.org/canister/ryjl3-tyaaa-aaaaa-aaaba-cai) is the smart contract that keeps track of the ICP Tokens, it has the Canister ID of `ryjl3-tyaaa-aaaaa-aaaba-cai`. The Canister ID is basically the address of the smart contract.

1. Install and import `Agent.js`

```jsx
import { Ed25519KeyIdentity } from "@dfinity/identity";
import { HttpAgent, Actor } from "@dfinity/agent";
import idlFactory from "./idlFactory.js";
```

1. Create a wallet instance or an “agent” instance

```jsx
// Create the identity from the hardcoded keys
const identity = Ed25519KeyIdentity.fromKeyPair(publicKeyDer, privateKey);
const principal = identity.getPrincipal();
const principalText = principal.toText();

// Output the Principal ID in text format
console.log("Principal ID (Text):", principalText);
// Create an HTTP agent with the identity
const agent = await HttpAgent.create({ identity });
```

2. Create the ICP Ledger Contract instance, or in ICP terms, contract is referred to as Actors

```jsx
// Ledger canister ID and actor creation
const ledgerCanisterId = "ryjl3-tyaaa-aaaaa-aaaba-cai"; // This is the main ledger canister ID

const actor = Actor.createActor(idlFactory, {
  agent,
  canisterId: ledgerCanisterId,
});
```

3. Function to build transaction data and send it to the IC network.
    
    ```jsx
    // Function to transfer ICP
    async function transferICP() {
      try {
    ```
    
    1. Determine the receiving account and amount
        
        ```jsx
        // Convert the string to a Principal object
            const ownerPrincipal = Principal.fromText("wyv54-vseyl-yofng-cgg2r-uns7s-jk47w-k7ygb-2gswo-mmqts-b4qls-dqe");
        
            // Create the Account object
            const account = {
              owner: ownerPrincipal,
              subaccount: [], // Optional: Specify subaccount if needed
            };
        
            // Call the account_identifier function from the actor
            const recipientUint8Array = await actor.account_identifier(account);
        
            // Convert Uint8Array to Hexadecimal string
            const recipient = uint8ArrayToHex(recipientUint8Array);
            
            // Amount to transfer: 1 ICP = 100_000_000 e8s
            const amount = BigInt(1 * 100_000);
        
        ```
        
    2. Build Transaction data and send it.
        
        ```jsx
            
        
            // Create the transfer argument
            const transferArgs = {
              memo: BigInt(0), // Optional memo, can be used to reference the transaction
              amount: { e8s: amount },
              fee: { e8s: BigInt(10_000) }, // Standard transaction fee of 0.0001 ICP (10,000 e8s)
              from_subaccount: [], // Optional, specify if using a subaccount
              to: recipient,
              created_at_time: [], // Optional, timestamp for the transaction
            };
        
            // Call the transfer method on the ledger canister
            const result = await actor.send_dfx(transferArgs);
            console.log("Transfer result:", result);
          } catch (error) {
            console.error("Error during transfer:", error);
          }
        }
        
        // Call the transfer function
        transferICP();
        ```
        
    

### What is the account identifier

The `account_identifier` method hashes our principal to create a unique hash output derived form the principal.

Calling `account_identifier` on an example principal `wyv54-vseyl-yofng-cgg2r-uns7s-jk47w-k7ygb-2gswo-mmqts-b4qls-dqe` will give us a unique 32 byte value `2f33a041bcbfceaff5a15170c43786bf1d131c08f0fcad7eee81cc0633f6d776`. This is the address of the account that we will transfer ICP to. The idea is that through different derivation paths, a single “principal” can have multiple “account identifiers”.

### Transaction confirmation

In Ethereum, we get a receipt for a confirmation, likewise, in ICP, the return value for a successful transfer is the Transaction block our transfer is included in.  The next section will discuss how we can verify the ICP Token Transaction.

## Verifying Transactions in Ethereum vs ICP

In Ethereum, the receipt we get after submitting the blocks tells us whether or not our transaction fails or is successfully included in the block. But having your transaction included in the block is not enough to be sure that your transaction is finalized, there might be a case where there is a temporary fork in the Ethereum network and the network eventually chooses the longest chain ( the chain your transaction is **not** in) and your transaction does not go through. Therefore, a transaction is generally considered finalized in Ethereum when there are 20 more blocks produced on top of the transaction your block is in. This is takes about 3 minutes.

To verify whether the ICP transaction has been finalized, we need not wait for 20 blocks. We can simply verify the the certificate of the block our transaction is in. 

In the Internet Computer Protocol (ICP), transaction finalization is significantly faster compared to Ethereum. The finalization process in ICP only takes about 2-3 seconds. This rapid finalization is due to the advanced consensus mechanisms employed by the ICP. From incorporating a state of the art cryptography technique which they call Chain-Key Technology. It allows the nodes in the network to “sign” information just like how an account is able to sign transactions. 

ICP’s Threshold Signatures allows the network to sign blocks while keeping their private key safe among the network. The network can sign pieces of information, therefore it can quickly certify an event is true and to verify it, we simply have to check the signature against its 48 bytes public key. This is what makes the ICP Network fast and trusted!

Next, using Agent.js, we’ll retrieve the our block transaction record, and verify the certificate.

 `Agent.Js`

```jsx
import { HttpAgent, Actor, Certificate } from "@dfinity/agent";
import { Principal } from "@dfinity/principal";
import idlFactory from "./idlFactory.js";

try {
  // Create a new HttpAgent instance
  const agent = await HttpAgent.create({ host: "https://ic0.app" });

  // Create an actor for the ledger canister
  const ledgerCanisterId = Principal.fromText("ryjl3-tyaaa-aaaaa-aaaba-cai");
  const ledgerActor = Actor.createActor(idlFactory, {
    agent,
    canisterId: ledgerCanisterId,
  });

  // Define the block query parameters
  const startBlock = BigInt(13201043);
  const length = BigInt(1);

  // Query the ledger for blocks
  const response = await ledgerActor.query_blocks({
    start: startBlock,
    length: length,
  });

  // // Log the entire response to the console
  // console.log("Block Query Response:", response);

  // Handle the certificate
  if (response.certificate) {
    const certBuffer = response.certificate[0];
    // console.log(certBuffer);

    await Certificate.create({
      certificate: certBuffer, // Pass the Uint8Array directly
      rootKey: agent.rootKey,
      canisterId: ledgerCanisterId,
    });

    console.log("Certificate is Valid");
  } else {
    console.error("No certificate found in response.");
  }
} catch (error) {
  console.error("Failed to query and verify block:", error);
}
```

