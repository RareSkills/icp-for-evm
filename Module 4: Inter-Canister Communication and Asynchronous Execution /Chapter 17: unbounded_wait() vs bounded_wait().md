# unbounded_wait() vs bounded_wait()

Functions can respond in milliseconds while others may take several seconds or longer, depending on how long the runtime takes to execute the function. 

For instance the function `loop_block()` below will take about **13** seconds to process since its computationally heavy, it loops 3.9 billion times.

`Canister B`

```rust
#[ic_cdk::update]
fn loop_block() -> u64 {
    let limit: u64 = 3_900_000_000; 
    let mut total: u64 = 0;

    for i in 0..limit {
        // Do something simple but not optimizable to a no-op
        total = total.wrapping_add(i ^ (i >> 3));
    }

    total
}

ic_cdk::export_candid!();
```

![Screenshot 2025-08-27 at 18.49.43.png](unbounded_wait()%20vs%20bounded_wait()/Screenshot_2025-08-27_at_18.49.43.png)

When another canister, **Canister A**, calls `loop_block()` (**Canister B**), it will need to wait for 13 seconds before resuming the function execution.

Code for **Canister A**:

```rust
use candid::Principal;
use ic_cdk::call::Call;

#[ic_cdk::update]
async fn call_loop_block(canister_b: Principal) -> u64 {

    let result = Call::unbounded_wait(canister_b, "loop_block").with_args(&(1 as u64,)).await;
    
    if result.is_ok() == false{
		    ic_cdk::trap("The inter-canister call failed");
    }
    
    let decoded_value = result.unwrap().candid::<u64>();
    
    if decoded_value.is_ok() == false {
		    ic_cdk::trap("Failed to decode data");
		}
    
    decoded_value.unwrap()
}

ic_cdk::export_candid!();
```

Overall, it about 15 seconds to process the inter-canister call:

![Screenshot 2025-08-27 at 18.51.24.png](unbounded_wait()%20vs%20bounded_wait()/Screenshot_2025-08-27_at_18.51.24.png)

One concern with using `Call::unbounded_wait()` is that if the `Canister B` canister deliberately stalls a response or never replies because the `Call` API failed to route the response, the function call from `Canister A` will never finish. 

The `Call::unbounded_wait()` constructor which we first introduced earlier will wait for however long to receive a response from the `Canister B` canister. During that “waiting” period, the original execution is paused and resumes only once the reply arrives, and the function caller would have to wait for a long time.

We would want to cut the reply short since waiting for too long is bad user experience. To solve this, ICP lets you enforce a maximum a time out on inter-canister calls through the `bounded_wait()` constructor.

## `Call::bounded_wait()` will only wait for a reply for certain time frame

Inter-canister calls with `Call::bounded-wait()` will wait for a default configuration of 300s to receive a reply from the called canister . If it doesn’t receive a response back in 300s, the function call will drop and return a *Call Deadline Expired*. Let’s demonstrate this.

We will configure the time out to 5 seconds, hence the response from the previous `Canister B` which takes 10 seconds, would not be waited for. 

Change `unbounded_wait` to `bounded_wait` and add the `.change_timeout` method. It takes an integer argument which represents the amount of seconds it would wait for the response.

```rust
use candid::Principal;
use ic_cdk::call::Call;

#[ic_cdk::update]
async fn b(canister_b: Principal) -> u64 {
    let result = Call::bounded_wait(canister_b, "loop_block")
        .change_timeout(5)
        .await;

    if result.is_err() {
        ic_cdk::api::trap(result.unwrap_err().to_string());
    }

    let decoded_result = result.unwrap().candid::<u64>();
    decoded_result.unwrap()
}

ic_cdk::export_candid!();

```

![Screenshot 2025-08-29 at 22.45.23.png](unbounded_wait()%20vs%20bounded_wait()/Screenshot_2025-08-29_at_22.45.23.png)

We’ll get a “*Call deadline has expired error”*, although the canister finishes executing. Since we only give the inter-canister call 5 seconds to return a response but it takes 13 seconds at least to respond. 

During the waiting period, the canister does not completely stop all execution, it will process other message that are in queue and resume the inter-canister call function when it receives a reply from the `Canister B` canister. We will discuss this in the next article.