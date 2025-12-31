# Decoding and Encoding Inter-Canister Arguments

When an inter-canister call succeeds and the function returns a value, we first need to unwrap `Ok(Response)` to get `Response`, which is a candid encoded. To we would have to deserialize it to obtain the value. Candid is the serialization format of ICP messages. We will use `.candid::<type>()` method to deserialize candid encoded messages.

`Canister B` has a new function `ret_val` which returns the fixed `u64` number **21**.

```rust
#[ic_cdk::update]
fn ret_val() -> u64 {
		21
}

ic_cdk::export_candid!();
```

Generate the Candid UI and deploy it.

![Screenshot 2025-08-29 at 15.09.35.png](Decoding%20and%20Encoding%20Inter-Canister%20Arguments/Screenshot_2025-08-29_at_15.09.35.png)

We’re going to decode the return value of `ret_val()`.

- First, unwrap the `Ok()` wrapper with the `.unwrap()` method, this would give us the raw candid encoded response.
- Second, we call the `.candid::<type>()` method on the unwrapped response. the “`type`" is set to the data type we want to decode it into.

In `Canister A` below, we have added the unwrapping and decoding logic to the inter_canister function .

```rust
use candid::Principal;
use ic_cdk::call::Call;

#[ic_cdk::update]
async fn inter_canister(_callee: Principal) -> u64 {

    let result = Call::unbounded_wait(_callee, "ret_val").await;
    
    let ok = result.is_ok();
    
    if ok == false {
        // Print the error if the call failed
        ic_cdk::api::trap( result.unwrap_err().to_string());
    }
    
	  // Unwraps Ok(Response) to Response
    let unwrapped_result = result.unwrap(); 
    // We now have the raw canid encoded response
    
    // Decodes the candid encoded raw response 
	  let decoded_value = unwrapped_result.candid::<u64>();
    
    // Check if we have successfully decoded it
    if decoded_value.is_ok() == false {
		    ic_cdk::trap("Failed to decode data");
		}
    
    // Unwrap Ok(decoded Value)
    decoded_value.unwrap() // returns the successfully decoded value
}

ic_cdk::export_candid!();
```

Call the inter-canister function and it should successfully return what `canister b` returns, 21.

![Screenshot 2025-12-31 at 17.46.21.png](Decoding%20and%20Encoding%20Inter-Canister%20Arguments/Screenshot_2025-12-31_at_17.46.21.png)

## **`.candid` returns a `Result` type as well:**

The result type that it returns is either an

- `Ok(decoded number: u64)` or an
- `Err(failed to decode the data)`.

We handle the error if decoding fails:  

```rust
if decoded_value.is_err{
    ic_cdk::trap("Failed to decode data");
}
```

and return the value if it succeeds: 

```rust
decoded_value.unwrap()
```

Let’s test this:

1. Deploy `Callee` and `Caller`.
2. Obtain `Callee`’s principal.
3. Call `inter_canister_call()` and pass `Callee`’s principal as its parameter.

![Screenshot 2025-08-25 at 22.48.12.png](Decoding%20and%20Encoding%20Inter-Canister%20Arguments/Screenshot_2025-08-25_at_22.48.12.png)

### **Failed Decoding data example**

`Callee`'s function `ret_val` returns a `u64`, but let’s try to decode the data into a `bool` by changing `.candid::<u64>` to `.candid::<bool>`, and the return value of the function to a `bool`.

```rust
use ic_cdk::call::Call

#[update]
async fn inter_canister_call(callee: Principal) -> bool {

    let result = Call::unbounded_wait(callee, "ret_val").await;
    
    if result.is_ok() == false{
		    ic_cdk::trap("The inter-canister call failed");
    }
    
    let decoded_value = result.unwrap().candid::<bool>(); // change here
    
    if decoded_value.is_ok() == false {
		    ic_cdk::trap("Failed to decode data");
		}
    
    decoded_value.unwrap()
}
```

The function fails with an error message: “Failed to decode data.”

![Screenshot 2025-08-28 at 16.42.59.png](Decoding%20and%20Encoding%20Inter-Canister%20Arguments/Screenshot_2025-08-28_at_16.42.59.png)

The error is caused in this line of code:

```rust
if decoded_value.is_ok() == false {
    ic_cdk::trap("Failed to decode data");
}
```

## **decode other data types using `.candid::<Type>()`**

If you are expecting the response to be of type `u64`, configure `.candid::<type>()` to `.candid::<u64>()`. 

If you are expecting a `String` response, then configure `.candid` to decode a `String`: **`.candid::<String>()`**

If you are expecting a `bool`: **`candid::<bool>()`**

## Encoding Arguments

Functions often require parameters, the sum() function below takes two `u64` numbers as its parameter arguments and returns their sum.

`Callee` 

```rust
#[ic_cdk::update]
fn sum(a: u64, b: u64) -> u64 {
    a + b
}

ic_cdk::export_candid!();
```

To pass arguments, use the  `.with_args()` method, it takes and place the fields inside a tuple reference, with each entry as the expected argument type in order.

`Caller` calls `sum`

```rust
use candid::Principal;
use ic_cdk::call::Call;

#[ic_cdk::update]
async fn inter_canister_call(callee: Principal) -> bool {

    let result = Call::unbounded_wait(callee, "sum")
        .with_args(&(1 as u64, 2 as u64))
        .await;

    if result.is_ok() == false{
		    ic_cdk::trap("The inter-canister call failed");
    }
    
    let decoded_value = result.unwrap().candid::<bool>(); // change here
    
    if decoded_value.is_ok() == false {
		    ic_cdk::trap("Failed to decode data");
		}
    
    decoded_value.unwrap()
}

ic_cdk::export_candid!();

```

![Screenshot 2025-08-29 at 00.59.04.png](Decoding%20and%20Encoding%20Inter-Canister%20Arguments/Screenshot_2025-08-29_at_00.59.04.png)

If the arguments don’t match up, it will trap with an error message of: “The Inter canister call failed”. Observe that we typecasted `1` to an `i32`.

```rust
use candid::Principal;
use ic_cdk::call::Call;

#[ic_cdk::update]
async fn inter_canister(callee: Principal) -> u64 {
    let result = Call::unbounded_wait(callee, "sum")
		    .with_args(&(1 as i32, 2 as u64)) // wrong argument type
			  .await;

    if result.is_ok() == false {
        ic_cdk::trap("The inter-canister call failed");
    }

    let decoded_value = result.unwrap().candid::<u64>(); // change here

    if decoded_value.is_ok() == false {
        ic_cdk::trap("Failed to decode data");
    }

    decoded_value.unwrap()
}

ic_cdk::export_candid!();
```

![Screenshot 2025-08-29 at 01.03.08.png](Decoding%20and%20Encoding%20Inter-Canister%20Arguments/Screenshot_2025-08-29_at_01.03.08.png)

In the next article we’ll learn when to use the `unbounded_wait()` or the `bounded_wait()` constructor when configuring your inter-canister call.