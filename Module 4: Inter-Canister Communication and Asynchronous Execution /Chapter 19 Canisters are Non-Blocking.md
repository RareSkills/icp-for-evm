# Canisters are Non-Blocking

When a canister makes an inter-canister call and waits for the response, it **does not block** or **halt execution**. Instead, the canister keeps executing **other transactions** until the inter-canister call returns.

In the diagram below, the canister starts processing **Tx 1** which performs an inter-canister call. While Tx 1 is waiting for the response from the `Call` API, the canister continues processing **Tx 2, Tx 3, and Tx 4** in sequence. Once the reply arrives, it resumes executing **Tx 1**.

![Screenshot 2025-11-20 at 20.45.36.png](../.gitbook/assets/Screenshot_2025-11-20_at_20.45.36.png)

These other transactions can be query or update calls, which can both **observe** and **modify** the state already committed by **Tx\_1**.

## Others Can Observe the Intermediate State

Consider canisters `A` and `B`. Canister `A` has a function `foo()` that increments the variable `x` twice. Once **before** making an inter-canister call to `B.bar()` and again after the inter-canister call returns.

```rust
thread_local!{
		static x : RefCell<u64> = RefCell::new(0);
}

async fn foo() -> bool {

    increment()

    // (2) non-blocking wait for B.bar()
    Call::bounded_wait("B", "bar").await;

    increment()
    
    return true;
}

#[ic_cdk::query]
fn increment() {
		X.with_borrow_mut(|cell| *cell +=1 );
}

#[ic_cdk::query]
fn get_x() -> u64 {
    X.with_borrow(|cell|*cell)
}
```

Canister `B` has a function `bar()` that is intentionally slow (it takes around 13 seconds to execute).

```rust
// This function will take about 13 seconds to execute
#[ic_cdk::update]
fn bar() {

		// Computationally heavy and slow
    let limit: u64 = 3_900_000_000; 
    let mut total: u64 = 0;

    for i in 0..limit {
        // Do something simple but not optimizable to a no-op
        total = total.wrapping_add(i ^ (i >> 3));
    }

    total
}
```

Now suppose **Alice** calls `A.foo()` when `X` is **0**.

* `foo()` increments `X` to `1`.
* Then it calls `B.bar()` and waits asynchronously for \~13 seconds.

During that waiting period, **Bob** can call `get_x()` and see the state that `foo()` has already committed. In this case, Bob’s `get_x()` call will return **1**.

![Screenshot 2025-11-20 at 20.49.11.png](../.gitbook/assets/Screenshot_2025-11-20_at_20.49.11.png)

Try to emulate the example above. Deploy canister `A` and `B`. Call `A.foo()`, then immediately call `get_x()` after, it will return **1**.

![Screenshot 2025-11-28 at 12.58.34.png](../.gitbook/assets/Screenshot_2025-11-28_at_12.58.34.png)

After `foo()` has returned `true`, call `get_x()` again, it will return **2**.

![Screenshot 2025-11-29 at 16.31.19.png](../.gitbook/assets/Screenshot_2025-11-29_at_16.31.19.png)

* The **first** `get_x()` executes during the waiting period of `foo()`. It returns **1** because `foo()` has already committed `X += 1`before the inter-canister call.
* The **second** `get_x()` executes after `foo()` finishes. It returns **2** because `foo()` increments `X` again after the inter-canister call.

On the other hand, if a canister processes transactions that modify its state while waiting for an inter-canister call response, the transaction may resume into **a different state** than the one it originally started off with.

## Others Can Modify The Intermediate State

Other transactions don’t just observe the intermediate state, they can **change** it.

In the earlier example, if Bob calls `increment()` instead while Alice’s `foo()` is waiting, he will have changed the canister state. When `foo()` resumes, it now runs against this **new** state.

![Screenshot 2025-11-20 at 20.50.03.png](../.gitbook/assets/Screenshot_2025-11-20_at_20.50.03.png)

Here’s what happens:

* Alice starts `foo()` when `X = 0`.
* `foo()` increments `X` to `1` and calls `B.bar()`.
* While `foo()` is waiting on `B.bar()`, Bob calls `increment()` and changes `X` from `1` to `2`.
* When `foo()` resumes, it now sees `X = 2` and increments it to `3`.

Functions that perform inter-canister calls **do not necessarily run against a single, unchanged canister state** from start to finish. The canister’s state can change while the function waits.

### Example of An Async Function Resuming Into a New Canister State

Let’s extend canister `A` with a simple check to detect whether `X` has been modified while `foo()` was waiting. We introduce two local snapshots:

* `x_before`: the value of `X` before the inter-canister call.
* `x_after`: the value of `X` after the inter-canister call.

If these two values differ, it means `X` changed while `foo()` was waiting, and `foo()` returns `false` early.

```rust
thread_local!{
		static x : RefCell<u64> = RefCell::new(0);
}

async fn foo() -> bool {

    increment()
    
    // value of x BEFORE inter-canister call
    let x_before = X.with_borrow(|cell|*cell);

    Call::bounded_wait("B", "bar").await;
    
    // value of x AFTER inter-canister call
    let X_after = X.with_borrow(|cell|*cell);
    
    // check that the X hasn't changed, return false if it has
    if X_before != X_after {
		    return false;
    }

    increment()
    
    return true;
}

#[ic_cdk::query]
fn increment() {
		X.with_borrow_mut(|cell| *cell +=1 );
}

#[ic_cdk::query]
fn get_x() -> u64 {
    X.with_borrow(|cell|*cell)
}

ic_cdk::export_candid!()
```

**Re-deploy** canister \*\*\*\*`A` and call `foo()`. While `foo()` is waiting on `B.bar()`, call `increment()`. When `foo()` resumes, it will observe that `x_before` is **not equal** to `x_after` and returns `false`.

![Screenshot 2025-11-28 at 13.02.59.png](../.gitbook/assets/Screenshot_2025-11-28_at_13.02.59.png)

The implications of non-blocking canisters affect how we code our asynchronous functions. The state may have changed while the function was waiting, so if a local variable captured a state variable before an inter-canister call, its value might be outdated when the function resumes.

## Design Implications of Non-Blocking Canisters

Because canisters are non-blocking, you should avoid patterns that implicitly assume: “The state will stay the same while I’m waiting for an inter-canister call.”

In particular, it’s dangerous to **read state**, perform an **inter-canister call**, and then **reuse the old state** to update your canister. For example, the `withdraw` function below stores the vault’s balance **before** an inter-canister call and reuses that value **after** the call to update the balance:

```rust
// Example vault contract

async fn withdraw(amount: u64) -> Result<(), Error> {

    // (1) Read the vault's balance
    let balance = get_balance();
    // check if they enough funds

    // (2) withdraw icp tokens to user
    Call::bounded_wait("icp_ledger", "transfer", amount).await;

    // (3) Use the old `balance` to calculate for the new balance
    set_balance(balance - amount);     // may be stale!
    
    Ok(())
}
```

During the inter-canister call (step 2), other messages can change the vault’s balance:

* another user might deposit,
* another withdrawal might already have reduced the balance,
* or an admin operation might have modified it.

When the function resumes, the `balance` variable may no longer match the canister’s actual balance, yet it is still being used to compute the new state.

Instead, we generally want to **re-check** or **recompute** state after the inter-canister call, or **anticipate** the state change and **explicitly roll it back** if needed.

### Anticipate and Recompute State

In the pattern below, we **anticipate** the state change before making the inter-canister call. We update the balance optimistically, then either:

* keep that state if the call succeeds, or
* **roll it back** if the call fails.

Because the balance may change while the canister is waiting, we also **re-read** the state when rolling back. That way, we only undo _our_ change and preserve any other concurrent updates.

```rust
// Anyone can withdraw
async fn withdraw(amount: u64) -> bool {

    // Read the vault's balance
    let balance = get_balance();
    
    // Anticipate the change
    set_balance(balance - amount);     

    // withdraw tokens to caller
    let result = Call::bounded_wait("icp_ledger", "transfer", amount).await;
    
    // explicitly handle the inter-canister call
	  if result.is_ok(){
		    // state-changes were already made, do nothing
		    // return ok
		    return true;
			  
	  } else{
			  // result is error
			  
			  // Read the new vault's balance
		    let new_balance = get_balance();
		    
		    // add the balance back to the vault
		    set_balance(balance + amount); 
			  
			  // return false indicating that the withdraw failed
			  return false;
	  }
}
```

This pattern acknowledges that **state changes may be committed before the await**, avoids relying on **stale local variables** after an inter-canister call, and correctly **recomputes and rolls back** state when the call fails.

One of the most complex concepts to grasp is ICP’s asynchronous execution model. Now that we have covered it, when performing an HTTP Outcall, which is an asynchronous System API that allows canisters to perform HTTP requests to web2 APIs, it also follows the pattern above.

In the next article, we’ll learn how to send Cycles between canisters, which is similar to sending Ether between Solidity contracts.
