# The Update Function: #[ic_cdk::update]

Functions that are annotated with the `#[ic_cdk::update]` attribute macro have state-changing permission. 

```rust
#[ic_cdk::update]
fn update_function() -> u8 {
		// state-changing logic
}
```

We’ll demonstrate modifying a canister’s state by creating a Counter canister and using an update function:  

### Counter Canister Example

The `counter` canister below contains:

- A mutable state variable, `static mut COUNTER` (unsafe).
- An update function, `increment()`, that increments `COUNTER` by one and returns the updated value.
- A query function, `get_counter()`, that returns `COUNTER`'s value.

Since `COUNTER` is declared with `static mut`, all access must be wrapped in `unsafe`.

```rust
// Mutable State Variable
static mut COUNTER: u128 = 0;

// View State
#[ic_cdk::query]
fn get_counter() -> u128 {
    unsafe {
        COUNTER
    }
}

// Modify State
#[ic_cdk::update]
fn increment() -> u128 {
    unsafe {
        COUNTER += 1;
        COUNTER
    }
}

// Candid Interface Export
ic_cdk::export_candid!();
```

Re-use the `hello_world` project to deploy the counter canister. Generate the candid Interface with:

```rust
generate-did hello_world_backend 
```

And run:

```rust
 dfx deploy. 
```

Then, call `increment()`, you’ll see that `COUNTER` is incremented. 

![Screenshot 2025-09-12 at 13.27.21.png](The%20Update%20Function%20#%5Bic_cdk%20update%5D/Screenshot_2025-09-12_at_13.27.21.png)

`COUNTER` is a mutable variable stored in the canister’s storage. Calling `increment()` updates that state by increasing `COUNTER` by one, while calling `get_counter()` reads and returns the `counter`’s value.

If by chance you use a query function instead of an update function to commit state changes, the query function will not throw a compile error like in Solidity. 

### State Changes Through Query Functions Do Not Revert

Using query functions to mutate canister state will **not revert**, and it will **not throw a compile error**. To illustrate this, in the previous `Counter` example, change the attribute macro of `increment()` from `#[ic_cdk::update]` to `#[ic_cdk::query]`:

```rust
static mut COUNTER: u128 = 0;

#[ic_cdk::query]
fn get_counter() -> u128 {
    unsafe {
        COUNTER
    }
}

#[ic_cdk::query] // Previously #[ic_cdk::update], now its query
fn increment() -> u128 {
    unsafe {
        COUNTER += 1;
        COUNTER
    }
}

ic_cdk::export_candid!();
```

The Rust code above will **not** throw a compile error.

Re-deploy `counter` canister and call `increment()`, it should return **1**. Then, call `get_counter()` and it returns **0**. `increment()` did not revert despite the attribute macro being query but the state-change to `COUNTER` wasn’t persisted.

![Counter example with query.png](The%20Update%20Function%20#%5Bic_cdk%20update%5D/Counter_example_with_query.png)

Query functions do not participate in consensus, which means the network does not agree on or record their effects. As a result, any state changes made inside a query function are **not saved**.

When `increment()` is marked as a query function, it *appears* to update `COUNTER` while the function is running, so the function can return **1**. However, this change is applied only to a **temporary copy** of the canister’s state. Once the query call finishes, that temporary state is discarded, and the original value of `COUNTER` remains unchanged.

### Conclusion

To keep the distinction clear, use `#[ic_cdk::update]` for state-changing operations, and `#[ic_cdk::query]` for state-viewing operations. 

In the next article, we’ll show how to write more idiomatic code for defining mutable state variables that doesn’t use the scary `unsafe` word.