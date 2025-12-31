# The Reverse Gas Model of ICP

This article explains the gas model of the Internet Computer Protocol and why it supports unsigned transactions as valid transactions.

On most blockchains such as Ethereum and Solana, gas fees are paid for by the user or a wallet. When a user calls a smart contract, the gas required to execute that function is deducted from the user’s balance. These fees are usually paid in the network’s native token — on Ethereum it’s ETH, on Solana it’s SOL.

On the Internet Computer Protocol, who the network charges for gas fees are reversed, they are charged **not** to the user, but to the canister that is being called.

## Canisters Pay For Gas Fees

When a user invokes a canister function, the gas fee is entirely charged to the canister that is called. This is ICP’s **reverse gas model**, where canisters pay for the gas fees of user transactions. To the user, the transaction appear free, behind the scenes, the canister covers the cost.

### The network accepts unsigned transactions

Since the network doesn't require users to pay for gas fees, it can accept unsigned transaction, identified by the anonymous principal — `2vxsx-fae`. The network doesn’t need the user’s signature and principal(address) to know whom to charge gas fees to — it always charges the called canister.

The caller’s unique identity is only relevant if the canister’s own logic cares about who’s calling (for example, to enforce authorization, prevent spam, or track ownership using `msg.caller()`). In which case if the canister disallows unsigned transactions from calling a function it can implement access control to reject calls coming from the anonymous principal.

## ICP’s Gas Token: `Cycles`

**Gas fees on the Internet Computer are not paid in `ICP` tokens**. Instead, the Internet Computer uses a special utility token called `Cycles`, which is used to pay for gas fees and storage.

Canisters pay for gas fees with the `Cycles` token. All canisters have a Cycles reserve to pay for gas fees which is tracked natively by the protocol.

We can verify for ourself that a canister has a `Cycles` balance with the following dfx command:

```jsx
dfx canister status <Canister-ID>
```

Deploy any canister locally. You’ll see a **Balance** section showing the amount of Cycles the canister has, which is typically around 3.5 trillion Cycles.

![Screenshot 2025-09-25 at 18.51.44.png](../.gitbook/assets/Screenshot_2025-09-25_at_18.51.44.png)

Each time a user calls one of the canister’s functions, the network deducts the gas fee from the canister’s Cycles balance.

Lets see for ourselves that when we call a function, the canister’s Cycles balance decreases due to the network deducting gas fees from it.

## Calling a Canister Consumes `Cycles`

The `fibonacci()` function below is computationally heavy and it consumes a lot of Cycles. We’ll verify that our canister’s `Cycles Balance` is reduced from the function call.

```rust
use ic_cdk::update;

#[update]
fn fibonacci(n: u64) -> u64 {
    if n <= 1 { n as u64 } else { burn(n - 1) + burn(n - 2) }
}

ic_cdk::export_candid!();
```

Run the `fibonacci()` function above **once** and pass **`43`** as the recursion.

![Screenshot 2025-10-30 at 04.33.55.png](../.gitbook/assets/Screenshot_2025-10-30_at_04.33.55.png)

Then, check the Cycles balance again with the command below, we should expect it to be reduced significantly.

```jsx
dfx canister status <Canister-ID>
```

![Screenshot 2025-09-25 at 18.58.46.png](../.gitbook/assets/Screenshot_2025-09-25_at_18.58.46.png)

`fibonacci(43)` has consumed roughly **500 billion** Cycles, almost 1/7th of the entire canister Cycles balance.

The user called the function, but the canister paid for it, with Cycles.

In traditional blockchains, users without gas tokens cannot call smart contracts. In ICP, if the canister do not have enough Cycles, it cannot run function calls from users.

### Low Cycles Balance Implication

When a canister runs out of Cycles, it stops executing update functions since it cannot pay for the gas fees.

Try running `fibonacci(43)` 6 more times to exhaust the canister’s Cycles balance. This operation will consume about 3 trillion cycles (500 billion \* 6). At the 6th time, `fibonacci(43)` will respond with an error: “canister is out of cycles”.

![Screenshot 2025-10-30 at 05.10.08.png](../.gitbook/assets/Screenshot_2025-10-30_at_05.10.08.png)

The call fails because the cost to run the function exceeds the amount of Cycles our canister has. We can’t run the function since the canister cannot pay for the computation cost.

Query functions however, does not consume cycles as it does not go through consensus and directly queries a node.

If we’re in a situation where our canister is out of Cycles, we’ll need to send more Cycles to the canister to allow it to service function calls. The next section discusses how Cycles are created and how we can top-up our canister with more Cycles.

## Cycles Origin

`Cycles` are created from converting `ICP` tokens into `Cycles` token — this is a one way conversion. The _Cycles Minting Canister_ (also an NNS system canister) facilitates this conversion— Users send `ICP` in, and receives `Cycles`, the gas token back.

### Convert `ICP` token into `Cycles` token

We can use the following dfx command to convert ICP tokens to Cycles tokens.

```jsx
dfx cycles convert --amount <AMOUNT>
```

Replace `<AMOUNT>` with **10** and it would convert about it to 35 Trillion Cycles.

```jsx
dfx cycles convert --amount 10
```

![Screenshot 2025-10-22 at 05.18.27.png](../.gitbook/assets/Screenshot_2025-10-22_at_05.18.27.png)

We have converted 10 ICP tokens into 17 Trillion Cycles, which we can send it to our canister that’s low on Cycles.

### The converted `Cycles` are tracked by the Cycles Ledger

The Cycles that a user gets from converting it from ICP is tracked on the _Cycles Ledger_, a system canister.

Like the ICP Ledger, which tracks the amount of ICP we have, the Cycles Ledger tracks the amount of Cycles we have. Across the mainnet and localnet they have the same Canister ID: `um5iw-rqaaa-aaaaq-qaaba-cai`.

We can use these Cycles to deploy a canister or send it to our canister that’s low on Cycles balance.

## Top up a canister with Cycles

The command below sends our Cycles from the Cycles Ledger to our canister of choice.

```jsx
dfx cycles top-up <To> <Amount>
```

Replace

* `<TO>` with the fibonacci canister: `uxrrr-q7777-77774-qaaaq-cai`
* `<AMOUNT>` with the amount of Cycles we converted, 17 Trillion Cycles: `17T`

```jsx
dfx cycles top-up uxrrr-q7777-77774-qaaaq-cai 17T
```

![Screenshot 2025-10-30 at 03.28.52.png](../.gitbook/assets/Screenshot_2025-10-30_at_03.28.52.png)

Verify the increase in Cycles balance with the command below.

```jsx
dfx canister status <Canister-ID>
```

**Before**: The fibonacci canister has less than 500 Billion Cycles

![Screenshot 2025-09-25 at 18.58.46.png](../.gitbook/assets/Screenshot_2025-09-25_at_18.58.46.png)

**After**: The fibonacci canister has about 100 quadrillion Cycles

![Screenshot 2025-08-23 at 15.19.07.png](../.gitbook/assets/Screenshot_2025-08-23_at_15.19.07.png)

The Internet Computer Protocol adopts the reverse gas model, where canisters pay for all the computation cost. The Cycles token is used as the gas token and to obtain it, we convert it from ICP tokens.

In the next article, we’ll discuss ICP’s storage rent model, where canisters pay for on-chain on a recurring basis with Cycles.
