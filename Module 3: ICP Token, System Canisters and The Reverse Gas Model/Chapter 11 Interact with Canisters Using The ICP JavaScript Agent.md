# Interact with Canisters Using The ICP JavaScript Agent Library

The _ICP JavaScript Agent_ library provides JavaScript abstractions to interact with the Internet Computer Protocol, including calling canister functions or querying on-chain information (like Ethers.js for Ethereum). Other agent libraries exist such as [Rust, Go, Dart, etc.](https://internetcomputer.org/docs/building-apps/interact-with-canisters/agents/overview)

This guide is a continuation of _Simple Token Canister_, with your `token_canister` already deployed, we’ll learn how to use ICP JavaScript Agent by interacting with `token_canister`.

### Outline:

* Install ICP Javascript Agent
* Connect To The Localnet
* Prepare the Canister’s Candid Interface
* Create a Canister Instance
* Query Calls
* Update Calls
* Signed transactions with an Identity

Create a new folder called `icp_javascript_agent` in your home directory and initialize a new npm project:

```bash
mkdir icp_javascript_agent
cd icp_javascript_agent
npm init -y
```

## Install ICP Javascript Agent

Install the ICP Javascript Agent package:

```jsx
npm i --save @dfinity/agent
```

Open `package.json` and change the `“type" : “commonjs"` to `“type" : “module"`. Node will treat your `.js` files as ES modules and allow you to use the `import`/`export` syntax required by `@dfinity/agent`.

![Screenshot 2025-07-08 at 18.01.08.png](../.gitbook/assets/Screenshot_2025-07-08_at_18.01.08.png)

Lastly, create a new javascript file called `agent.js`. We’ll write our script inside of `agent.js`.

![Screenshot 2025-08-21 at 14.11.04.png](../.gitbook/assets/Screenshot_2025-08-21_at_14.11.04.png)

## Connect To The Local IC Instance

To start off, we’ll instantiate an **HTTP agent** that manages all communication between your JavaScript code and the local ICP replica.

### Create an HTTP agent and Verify the connection

Import the `HttpAgent` class and call its `.create()` method with the localnet URL `http://127.0.0.1:4943`.

```rust
import { HttpAgent } from '@dfinity/agent';

const agent = await HttpAgent.create({
  host: 'http://127.0.0.1:4943',
});
```

Next, call `agent.status()` to check that the replica is running and reachable:

```jsx
import { HttpAgent } from '@dfinity/agent';

const agent = await HttpAgent.create({
  host: 'http://127.0.0.1:4943',
});

//new code
const status = await agent.status();
console.log(status);
```

Run the script with:

```rust
node agent.js
```

If the connection succeeds, the output will include `replica_health_status: "healthy"`.

![Screenshot 2025-08-21 at 14.40.24.png](../.gitbook/assets/Screenshot_2025-08-21_at_14.40.24.png)

Now that we have an established connection with the localnet, we can interact with `token_canister` canister as long as we have its:

* &#x43;_&#x61;nister Id, and_
* _Candid Interface_

The next section explains how to obtain `token_canister`’s Candid interface in javaScript format.

## Import `Token_Canister`'s Candid Interface

This section describes how we can do obtain a canister’s _Candid Interface_ in javascript through dfx.

Head over to `*token_canister*` directory and run the command below. This will translate the canister’s Candid Interface into javascript, under the file: `token_canister_backend.did.js`

```rust
dfx generate token_canister
```

The translated candid interface file, `token_canister_backend.did.js`, can be found at:

```rust
token_canister
		/src
				/declarations
						/token_canister_backend
								/**token_canister_backend.did.js**
```

Copy `token_canister_backend.did.js` over to **`icp_javascript_agent`** folder.

![Screenshot 2025-08-21 at 14.30.00.png](../.gitbook/assets/Screenshot_2025-08-21_at_14.30.00.png)

Then in the `agent.js` file, import `idlFactory` from `token_canister_backend.did.js` file.

```jsx
import { HttpAgent } from '@dfinity/agent';
import { idlFactory } from './token_canister_backend.did.js';

const agent = await HttpAgent.create({
  host: 'http://127.0.0.1:4943',
});
```

`idlFactory` is an exported function of the candid interface.

Now that we’ve established a connection to the localnet and obtained the Candid interface for `token_canister`, we can start interacting with the canister.

## Create a Canister Instance

Import `Actor` from the `@dfinity/agent` library. The `Actor` object creates a callable instance of a canister.

```rust
import { Actor } from '@dfinity/agent';
```

`Actor` accepts three parameters:

1. The candid interface, which we have imported as `idlFactory`
2. The agent, which was created using `HttpAgent`.
3. The canister ID of `token_canister`, which we can get e.g. `vu5yx-eh777-77774-qaaga-cai`

Here’s the complete code example of an `Actor` canister instance object.

```jsx
import { HttpAgent } from '@dfinity/agent';
import { idlFactory } from './token_canister_backend.did.js'; //imports candid interface
import { Actor } from '@dfinity/agent';

const agent = await HttpAgent.create({
  host: 'http://127.0.0.1:4943',
});

const TOKEN_CANISTER_ID = 'ufxgi-4p777-77774-qaadq-cai';

// Create the actor with the agent and the canister ID, and candid interface
const token_actor = Actor.createActor(
  idlFactory,
  {
    agent,
    canisterId: TOKEN_CANISTER_ID,
  }
);

// query calls or update calls
```

The subsequent section are examples of calling `token_canister` functions.

## Query Calls

Now that we have the agent properly configured to interact with our canister, let’s start the interaction by retrieving the metadata information via query calls. Query calls in ICP are free, just like view calls in Solidity.

### **Retrieving token `name`**

The function below invokes the `name()` function of the `token_canister`:

```tsx
// copy paste code found in ##Create Canister Instance

const name = await token_actor.name();
console.log(res);
```

![Screenshot 2025-08-21 at 15.16.01.png](../.gitbook/assets/Screenshot_2025-08-21_at_15.16.01.png)

### **Retrieving the token `symbol`**

`get_symbol()` returns a string representing the symbol of the token canister.

```tsx

const symbol = await token_actor.symbol();
console.log(symbol);
```

![Screenshot 2025-08-21 at 15.16.36.png](../.gitbook/assets/Screenshot_2025-08-21_at_15.16.36.png)

### Retrieving the `token_supply`

```tsx
const total_supply = await token_actor.token_supply();
console.log(total_supply);
```

## Update Calls

Let’s start by invoking a state-changing function, `mint()`.

Here’s the code which would interact with the `mint()` function, and supply its arguments, the receiver principal and the amount of tokens to mint.

```tsx
import { Principal } from '@dfinity/principal';

async function mint_tokens(to_id: string, amount: bigint | number) {
  const to = Principal.fromText(to_id);
  const ok = await token_actor.mint(to, BigInt(amount));
  return ok; // true if mint succeeded
}

// Example
const recipient = 'aaaaa-aa'; // replace with a real principal
const mint_result = await mint_tokens(recipient, 5000);
console.log('Mint result:', mint_result);
```

Run the code with

```rust
node agent.js
```

You should expect a Failure return statement “only owner can call this function”:

![Screenshot 2025-08-21 at 15.39.03.png](../.gitbook/assets/Screenshot_2025-08-21_at_15.39.03.png)

The reason the mint function failed above is because we have not configured ICP Javascript Agent to using the identity which deployed the canister.

We can check our Principal by calling `agent.getPrincipal()`

```rust
const principal = await agent.getPrincipal();
const currentIdentity = principal.toText();

console.log('Current identity principal (agent):', currentIdentity);
```

The output is as follows:

![Screenshot 2025-08-21 at 16.03.16.png](../.gitbook/assets/Screenshot_2025-08-21_at_16.03.16.png)

If we do not attach an identity, all calls from the script is classified to have come from the anonymous principal.

We can export the registered owner’s identity from dfx and use it in the `icp_javascript_agent`, here’s how.

## Signed Transactions from ICP Javascript Agent

In order to call `mint()`, ICP Javascript Agent needs to use the identity that dfx used to deploy the canister. We can use the identity by exporting the private keys from dfx and importing it into ICP Javascript agent.

### Export dfx identity

In your terminal run:

```rust
dfx identity export
```

This would export your private key, it looks like this:

![Screenshot 2025-08-21 at 16.06.55.png](../.gitbook/assets/Screenshot_2025-08-21_at_16.06.55.png)

### Import the private key

Within your icp\_javascript\_agent folder, create a new file called `owner.pem` and copy paste the private key that we just exported from dfx. `.pem` is a file format for storing private keys.

`owner.pem`

```
-----BEGIN EC PRIVATE KEY-----
MHQCAQEEICJxApEbuZznKFpV+VKACRK30i6+7u5Z13/DOl18cIC+oAcGBSuBBAAK
oUQDQgAEPas6Iag4TUx+Uop+3NhE6s3FlayFtbwdhRVjvOar0kPTfE/N8N6btRnd
74ly5xXEBNSXiENyxhEuzOZrIWMCNQ==
-----END EC PRIVATE KEY-----
```

We’ll need to install the cryptographic library that…:

```jsx
npm i @dfinity/identity-secp256k1
```

import the library:

```jsx
import { Secp256k1KeyIdentity } from '@dfinity/identity-secp256k1';
```

Get the current Identity

```jsx
// Get the current principal from the agent
const principal = await agent.getPrincipal();
const currentIdentity = principal.toText();

console.log('Current identity principal (agent):', currentIdentity);
```

The code above should return the same principal as the owner as long as you have exported the same identity that you used to deploy token\_canister.

Now that we have set our identity to be the owner’s identity we can mint tokens to any Principal.

### **Call `mint()` with the owner’s identity**

The code below sets up the identity from the pem key file, it connects to the local net, creates an Actor canister instance and calls the mint function to mint itself 1000 tokens.

```jsx
import { HttpAgent } from '@dfinity/agent';
import { Actor } from '@dfinity/agent';
import { idlFactory } from './token_canister_backend.did.js';
import { Secp256k1KeyIdentity } from '@dfinity/identity-secp256k1';
import { Principal } from "@dfinity/principal";
import fs from 'fs';

// Load identity
const pemKey = fs.readFileSync("owner.pem", "utf8");

// create identity from pem key file
const identity = Secp256k1KeyIdentity.fromPem(pemKey);

const agent = await HttpAgent.create({
    host: 'http://127.0.0.1:8080',
    identity,
});

const TOKEN_CANISTER_ID = 'vu5yx-eh777-77774-qaaga-cai';

const token_actor = Actor.createActor(idlFactory, { agent, canisterId: TOKEN_CANISTER_ID, });

async function mint_tokens(to_id, amount) {
    const to = Principal.fromText(to_id);
    const ok = await token_actor.mint(to, BigInt(amount));
    return ok; // true if mint succeeded 
}
const recipient = 'hpikg-6exdt-jn33w-ndty3-fc7jc-tl2lr-buih3-cs3y7-tftkp-sfp62-gqe';
// replace with a real principal 
const mint_result = await mint_tokens(recipient, 5000);
console.log('Mint result:', mint_result);
```

Expected output:

```bash
Mint result: true
```

### Check balance of a Principal

The `get_balance()` function below can be added to the code above to query the balance of the principal we have minted. We set the arguments to the canister principal in string form.

```tsx
import { Principal } from '@dfinity/principal';

async function get_balance(account_id: string) {
  const p = Principal.fromText(account_id);
  const bal = await token_actor.balance_of(p);
  console.log(bal.toString()); // Nat -> BigInt
}

// Example
await get_balance('aaaaa-aa');
```

## `Transfer()` function call

Since we’ve minted it to ourselves, we can create a function that calls the transfer function and send RareSkills token to any other principal.

```tsx
import { Principal } from '@dfinity/principal';

async function transfer_tokens(to_id, amount) {
  const to = Principal.fromText(to_id);
  const ok = await token_actor.transfer(to, BigInt(amount));
  return ok; // true if transfer succeeded
}

// Example
const transfer_result = await transfer_tokens('aaaaa-aa', 1000);
console.log('Transfer result:', transfer_result);
```

If you want to test it, set the parameters of `transfer_tokens` to `transfer_tokens(your principal of choice, amount of choice).`

We believe to have given enough examples to use ICP Javascript Agent. An exercise we have for you is to call the approve and transfer-from function yourself.

* `Approve(spender, amount)`. The caller of the function agrees that the spender can manage `amount` amount of tokens in behalf of the caller.
* `ALLOWANCE()` is where the token delegation information is stored
* `Transfer_from(owner, to , amount)` allows the spender to transfer the owner’s tokens.

## Exercise: Thief Thiel

`Default` has entrusted Thiel to spend his token wisely, however, Thiel has ulterior motives and wants to take all of his tokens for himself. `Default` mints 1000 tokens and approves thiel to spend all of it.

Create another dfx idenitty and name it `Thiel`. Then, re-enact the case scenario above as `Thiel` and steal `Default`’s tokens.

Token balances before:

* `Default` : 1000 RareSkills Tokens
* `Thiel` : 0 RareSkills Tokens

Token balances after:

* `Default` : 0 RareSkills Tokens
* `Thiel` : 1000 RareSkills Tokens
