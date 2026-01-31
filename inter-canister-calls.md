# Inter-Canister Calls

Smart contracts often need to interact with other contracts to share liquidity, compose with external protocols, or access shared infrastructure. In this article, we’ll learn how canisters can call functions from another canister. 

![Untitled-2025-06-09-1626.excalidraw-5.png](inter-canister-calls/untitled-2025-06-09-1626.excalidraw-5.png)

## The `Call` System API

On the Internet Computer, a canister invokes a function on another canister via the **Call System API**. In Rust canisters, this API is exposed through the Call type in the `ic_cdk::call` module:

```rust
use ic_cdk::call::Call;
```

`Call` is a **builder struct** that lets you configure and execute an inter-canister call. To create a `Call`, you can use one of two constructor functions:

- `Call::unbounded_wait(...)`
- `Call::bounded_wait(...)`

Both constructors take two arguments:

- the **target canister principal** (`Principal`), and
- the **method name** (the function you want to call on that canister).

Here’s what constructing an inter-canister call looks like:

- **`Call::unbounded_wait()`:**
    
    ```rust
    use ic_cdk::call::Call;
    
    async fn inter_canister_call() {
    
    		// Unbounded Wait
    		let result = Call::unbounded_wait("target_canister", "function_name").await;
    }
    ```
    
- **`Call::bounded_wait()`:**
    
    ```rust
    use ic_cdk::call::Call;
    
    async fn inter_canister_call() {
    		
    		// Bounded Wait
    		let result = Call::bounded_wait("target_canister", "function_name").await;
    }
    ```
    

The difference lies in their **wait policy**:

- `unbounded_wait` waits **indefinitely** for the callee’s response.
- `bounded_wait` waits for a **bounded amount of time**, after which the call fails if no response is received.

We’ll revisit this distinction later.

After constructing a Call, you can further configure it using builder methods such as:

- `.with_args(...)` to attach arguments
- `.with_cycles(...)` to send Cycles to the callee

## Inter-canister calls are asynchronous

Inter-canister calls execute **asynchronously**. This means that once a call is issued, it is handled independently by the runtime and executed in a **separate transaction** from the caller’s. The broader implications of asynchronous execution and its effects on state changes will be discussed in the next article.

### `.await` and `async`

In Rust, calling an asynchronous function or System API must be followed by the `.await` keyword. Without `.await`, the function will not actually be executed. Therefore, `Call::unbounded_wait()` **must** be followed by `.await` to call the function as shown in the code snippet below.

```rust
use ic_cdk::call::Call;

#[ic_cdk::update]
fn inter_canister_call() {

		//*** .await keyword added ***
		let result = Call::unbounded_wait(target_canister, "function_name").await;
}
```

Because `.await` is used, the function that calls `Call::unbounded_wait()` **must** be declared as `async` as shown in the example below:

```rust
use ic_cdk::call::Call;

// *** async keyword added ***
#[ic_cdk::update]
async fn inter_canister_call() {

		//*** .await keyword added ***
		let result = Call::unbounded_wait(target_canister, "function_name").await;
}
```

## **The `Call` System API Returns** a `Result`

The response that the `Call` API returns will be a `Result` type. Specifically, `Ok(Response)` or `Err(CallFailed)`.

- `Ok(Response)` means the inter-canister call is successful. `Response` is a raw candid encoded value returned by the remote function.
- `Err(CallFailed)` means that the inter-canister call failed. `CallFailed` is an error type containing a rejection code and a message that describes **why** the call failed. Details of this error type will be discussed later.

### A Simple Check for an `Ok(Response)` or `Err(CallFailed)`

To simply identify whether the inter-canister call succeeded, we can use the `.is_ok()` method. It returns:

- `true` if the `Result` is `Ok(Response)`,
- `false` if it is `Err(error)`.

Here’s a code snippet that shows how to use `.is_ok()` on `result`.

```rust
#[ic_cdk::update]
async fn inter_canister(_Callee: Principal) -> bool {

		let result = Call::unbounded_wait(_Callee, "function_name").await;
		
		let ok = result.is_ok();
		
		ok // returns true or false
}
```

In Solidity, this is similar to a low-level contract call. The example below shows a low-level call that plays a role similar to `Call::unbounded_wait()`. The `bool ok` value is analogous to calling `.is_ok()` in our Rust code: it indicates whether the call succeeded.

```rust
function callFunction(address target) external returns (bool) {
    (bool ok, bytes memory ret ) = target.call(
        abi.encodeWithSignature("function_name()")
    );
    
    return ok;
}
```

To demonstrate how inter-canister calls work in practice, we’ll use a **minimal example** built with `Call::unbounded_wait()` to invoke a function on another canister. We’ll then progressively extend the example by adding arguments, sending Cycles, handling errors, and decoding return values.

## Simple Inter-Canister Call Example

In this section we’ll create an example of a successful inter-canister call.

We’ll begin by creating **Canister B**, which will act as the callee. This canister exposes a single update method, `do_nothing`, which neither modifies state nor returns a value.

```rust
#[ic_cdk::update]
fn do_nothing() {
// actually do nothing
}

ic_cdk::export_candid!();
```

Deploy `Canister B` and note its **Canister ID** (for example: `u6s2n-gx777-77774-qaaba-cai`). We’ll need this principal when calling it from Canister A.

![Screenshot 2025-08-28 at 15.22.58.png](inter-canister-calls/bb1944ae-aeda-4d26-a9f1-1c1385219a99.png)

Next, we create **Canister A**, which performs the inter-canister call.

```rust
use candid::Principal;
use ic_cdk::call::Call;

#[ic_cdk::update]
async fn call_do_nothing(canister_b: Principal) -> bool {
		let result = Call::unbounded_wait(canister_b, "do_nothing").await;
		result.is_ok()
}

ic_cdk::export_candid!();
```

In above canister, the `call_do_nothing()` function does the following:

1. Calls `do_nothing` on `Canister B` using `Call::unbounded_wait()`.
2. Awaits the result.
3. Returns `true` if the call succeeded, or false otherwise.

Deploy Canister A, then invoke `call_do_nothing`, passing **Canister B’s principal** as the argument.

![Screenshot 2025-08-28 at 16.10.48.png](inter-canister-calls/screenshot-2025-08-28-at-16.10.48.png)

When `call_do_nothing` is executed:

1. `Call::unbounded_wait(canister_b, "do_nothing").await` returns a value of type
`Result<Response, CallFailed>`.
2. `.is_ok()` checks whether the result is wrapped in `Ok(...)`
    1. Returns true for `Ok(Response)`
    2. Returns false for `Err(CallFailed)`

A successful inter-canister call always yields `Ok(Response)`, where `Response` contains the raw return value of the called function (empty in this example).

## Failed Inter-canister Call Example: `Err(CallFailed)`

Upon setting the wrong canister ID or an invalid method, the result that the `Call` API returns will be of type `Err(CallFailed)`. The `CallFailed` payload will contain one of the following reasons:

1. The canister ID or the method doesn’t exist.
2. The canister trapped.
3. The Cycles balance of the target canister is insufficient to execute your inter-canister call.
4. The network failed to route your call (cases of such are rare).

In Canister `A`, slightly change the target canister’s principal so that it’s wrong. `call_do_nothing` would return **false**.

![Screenshot 2025-08-28 at 17.51.33.png](inter-canister-calls/screenshot-2025-08-28-at-17.51.33.png)

What’s happening is that `Call::unbounded_wait()` returns an error, `Err(CallFailed)`. `.is_ok()` returns `false` if the response is an `Err(CallFailed)`.

### Inspecting the `CallFailed` message

We can unwrap `Err(CallFailed)` to extract the underlying error message and gain a clearer understanding of **why** the inter-canister call failed, as shown below:

```rust
use candid::Principal;
use ic_cdk::call::Call;

#[ic_cdk::update]
async fn inter_canister(_callee: Principal) -> bool {

    let result = Call::unbounded_wait(_callee, "do_nothing").await;

    let ok = result.is_ok();

    if ok == false {
        // Print the error if the call failed
        ic_cdk::api::trap(result.unwrap_err().to_string());
    }

    true
}

ic_cdk::export_candid!();
```

In the above code, we:

1. Perform an inter-canister call using `Call::unbounded_wait(...)` and await its result.
2. Check whether the call succeeded using `.is_ok()`.
3. If the call failed, unwrap the `Err(CallFailed)` value and convert it into a string.
4. Trap with that message, causing the function to panic and revert while surfacing the precise reason for failure.

Re-deploy the canister and call the `call_do_noting()` function. You will observe that the inter-canister call fails with the error message: **“Canister has no update method `do_nothing()`.**

![Screenshot 2025-08-29 at 15.06.48.png](inter-canister-calls/screenshot-2025-08-29-at-15.06.48.png)

Let’s look at another error scenario where the failure originates from **Canister B**, specifically due to its function trapping.

To achieve this, replace the implementation of `do_nothing()` in Canister B with an intentional trap:

```rust
// Canister B

#[ic_cdk::update]
fn do_nothing() {
		ic_cdk::api::trap("Intentional Trap");
		
}

ic_cdk::export_candid!();
```

Re-deploy `Canister B`, then call `call_do_nothing` again from `Canister A`, passing `Canister B`’s principal as before. You will observe that the inter-canister call now fails with an error message containing **“Intentional Trap”**, which is propagated from the callee canister.

![Screenshot 2025-08-29 at 00.46.31.png](inter-canister-calls/9e293233-28b8-461e-a35a-eac1e14b46da.png)

The error message that the called canister returns is part of the `Err(CallFailed)` result of the inter-canister call.

### Wrapping Up

In this article, we explored the fundamentals of **inter-canister calls** on the Internet Computer using the Call System API. We learned how canisters invoke functions on other canisters asynchronously, how to await and inspect the results of those calls, and how success and failure are represented through `Ok(Response)` and `Err(CallFailed)`.

Through a minimal example, we demonstrated how `Call::unbounded_wait()` can be used to perform a basic inter-canister call and how `.is_ok()` provides a simple way to detect whether the call succeeded. We also examined common failure scenarios, including invalid method names and traps originating from the callee canister.

In the next article, we’ll learn how to pass arguments to inter-canister calls and how to decode the values returned by the called function.