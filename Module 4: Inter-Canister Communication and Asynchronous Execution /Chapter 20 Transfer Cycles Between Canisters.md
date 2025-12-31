# Transfer Cycles Between Canisters

Cycles can be sent directly from one canister to another as part of an inter-canister call using the `.with_cycles()` method.

A canister might require that you attach cycles to cover their execution cost or to add economic disincentive towards malicious actors draining the canister’s cycles.

### `.with_cycles()` sends Cycles to the Called Canister

The `.with_cycles(amount)` method for `Call` lets a canister attach a specified `amount` of Cycles to another canister through an inter-canister call. The specified `amount` of Cycles is taken out of the **caller canister’s** Cycles balance and sent to the called canister. In the code example below,

```rust
#[ic_cdk::update]
async fn attach_cycles(_callee: Principal){

		// attach cycles
		let result = Call::unbounded_wait(_callee , "accept_cyles")
												.with_cycles(amount)
												.await
}
```

This is analogous to Solidity’s low-level `.call` with `value`:

```solidity
function attachEtherRaw(address payable callee) external payable {
    (bool ok, ) = callee.call{value: msg.value}(
        abi.encodeWithSignature("acceptEther()")
    );
    require(ok, "call failed");
}
```

Attaching cycles to an inter-canister call is not enough by itself. The receiving canister function must explicitly accept those Cycles; otherwise, the Cycles are refunded to the caller.

### Receive Cycles with `msg_cycles_accept()`

For another canister to receive Cyles through an inter-canister call, the receiving function needs to call `msg_cycles_accept()` to accept the Cycles. This is similar to declaring a `payable` function in Solidity.

```rust
#[ic_cdk::update]
fn receive() -> u128 {
		ic_cdk::api::msg_cycles_accept(//u128 number)
}
```

`msg_cycles_accept()` takes a **u128** number which represents the amount of the Cycles it would accept and returns the amount of Cycles it accepted.

If the receiving function doesn’t implement `msg_cycles_accept`, it would not accept the cycles.

### Simple POC of Sending Cycles

The `callee` canister below has a `receive()` function that accepts Cycles and returns the amount of Cycles it accepted. We use the System API `msg_cycles_available()` (like `gasleft()` in solidity) to accept all of the Cycles that is attached to the inter-canister call.

```rust
use ic_cdk::api::{msg_cycles_accept, msg_cycles_available};

#[ic_cdk::update]
fn receive() -> u128 {
    msg_cycles_accept(msg_cycles_available())
}

ic_cdk::export_candid!();
```

The `caller` canister below attaches an amount of cycles the user specified.

```rust
#[ic_cdk::update]
async fn receive(_callee: Principal, amount: u128) {
		let raw_result = Call::unbounded_wait(_callee , "receive")
												.with_cycles(amount)
												.await;
												
		let decoded_result = raw_result.unwrap().candid::<u64>();
		decoded_result.unwrap()
}
```

Deploy both canisters and call `receive()`, the `callee` accepted exactly the amount we specified.

![Screenshot 2025-08-27 at 01.44.14.png](../.gitbook/assets/Screenshot_2025-08-27_at_01.44.14.png)

### What happens if receiver does not implement `msg_cycles_available()`?

Try removing `msg_cycles_accept` and only return `msg_cycles_available`.

`Callee`:

```rust
use ic_cdk::api::{msg_cycles_accept, msg_cycles_available};

#[ic_cdk::update]
fn receive() -> u128 {
    msg_cycles_available() // new code here
}

ic_cdk::export_candid!();
```

Then in the `Caller` canister, return `msg_cycles_refunded()`. the api `msg_cycles_refunded` tells us how much of the Cycles were refunded from the inter-canister call.

`Caller`

```rust
use candid::Principal;
use ic_cdk::call::Call;
use ic_cdk::api::msg_cycles_refunded;

#[ic_cdk::update]
async fn attach_cycles(_callee: Principal, amount: u128) -> u128 {
    let raw_result = Call::unbounded_wait(_callee, "accept_cycles").with_cycles(amount).await;
		// we did not parse the data as we only want to see the amount of cycles refunded
    msg_cycles_refunded()
}

ic_cdk::export_candid!();
```

Re-deploy the callee canister and call `attach_cycles` from the caller canister.

![Screenshot 2025-08-27 at 01.50.01.png](../.gitbook/assets/Screenshot_2025-08-27_at_01.50.01.png)

We can see that the cycles was fully refunded because the called canister function does not implement `msg_cycles_accept()`.

To send cycles, use the `.with_cycles` along with the inter-canister call. The receiving function needs to implement `msg_cycles_accept(amount)` or it will not accept the cycles.

## Handling Cycles Drainage with a Minimum Fee

One of the main concerns for canisters is that **they pay for execution in cycles**. To avoid having our cycles balance drained by expensive or abusive calls, we can require callers to attach cycles to the call. The canister then accepts only the amount it wants to keep, and any extra cycles are automatically refunded to the caller by the protocol. Here’s how to implement that pattern:

## Charge A Minimum Fee

Callers **must attach at least `MIN_FEE` cycles**. The canister accepts only `MIN_FEE`. Anything extra is **automatically refunded** to the caller.

```rust
use ic_cdk::api::call::{msg_cycles_available128, msg_cycles_accept128};

const MIN_FEE: u128 = 10_000_000_000_000; // e.g. 10T cycles, adjust to your needs

#[ic_cdk::update]
fn expensive_with_min_fee() {
    // 1. How many cycles did the caller attach?
    let available = msg_cycles_available128();

    if available < MIN_FEE {
        ic_cdk::trap("Not enough cycles attached for this call");
    }

    // 2. Accept only the minimum fee we want to keep
    let accepted = msg_cycles_accept128(MIN_FEE);
    assert_eq!(accepted, MIN_FEE);

    // do work
}

```

**What’s happening:**

* Caller must attach `≥ MIN_FEE` cycles.
* Canister only accepts `MIN_FEE`, so that becomes your “fee pot”.
* Extra cycles are refunded automatically at the end of the call.

This creates a **hard economic barrier**: spamming this function costs real cycles.
