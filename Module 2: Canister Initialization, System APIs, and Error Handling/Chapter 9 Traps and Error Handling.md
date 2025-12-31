# Traps and Error Handling

This article discusses how to revert (trap) functions and handle logic or runtime errors in Rust canisters.

Ethereum developers typically guard against errors by using revert checks that abort execution and roll back state changes. For example:

```solidity
contract RevertExample {

    function safeDivide(uint256 a, uint256 b) public pure returns (uint256) {
        require(b != 0, "division by zero"); // reverts if b is 0
        return a / b;
    }
    
}
```

On ICP, the equivalent of reverting a function is to call the `trap("error message")` API.

## Reverting Execution in Rust Canisters

With that comparison in mind, let’s look at how execution is aborted in Rust canisters on the Internet Computer.

### **Reverting with `trap()`**

`trap()` is a System API that tells the runtime to abort execution and roll back the state changes made by the current call (this is only true if the function call does not perform an inter-canister call).

```rust
use ic_cdk::api::trap;
```

`trap("error message")` takes a string slice (`&str`) as its argument. When the trap is triggered, execution immediately aborts and the provided message is returned to the caller as the trap error.

```rust
use ic_cdk::api::trap;

#[ic_cdk::update]
fn safe_divide(a:u64, b:u64) -> u64 {

    if b==0 {
        trap("division by zero");
    }
		
    return a / b;
}

ic_cdk::export_candid!();
```

Deploy the canister above and assign argument `b` as **0**. The function would return an error along with the custom trap error message.

![Screenshot 2025-12-04 at 20.30.47.png](../.gitbook/assets/Screenshot_2025-12-04_at_20.30.47.png)

## Revert with the `assert!()`

We can also revert a function by using the `assert!()` macro, which causes a panic at runtime if the defined conditions are violated. Within the `safeDivide` function below, we replaced the `trap()` API with the `assert!()` macro

```rust

#[ic_cdk::update]
fn safe_divide(a:u64, b:u64) -> u64 {
		// revert
		assert!(b!=0, "division by zero");
		
    return a / b;
}

ic_cdk::export_candid!();
```

Deploy the canister above and call `safeDivide()`, you would also expect to see the “division by zero” error message.

![Screenshot 2025-12-04 at 20.22.21.png](../.gitbook/assets/Screenshot_2025-12-04_at_20.22.21.png)

A panic at runtime under the hood causes the canister to `trap()`. Therefore using both traps and asserts are acceptable, the utilization of `trap()` or `assert!()` depends on your preference.

### **Limitations Associated with Reverts**

Reverting transactions with `trap()` can be tricky. In the case where the function call involves calling another canister (inter-canister calls), a revert would only **partially rollback** some of the state-changes that were made (the details are discussed in the fourth module). This could leave our canister in an awkward situation where only some of the state-changes were rolled back, and some, permanently committed.

## **Returning Errors Instead of Reverting**

To avoid these pitfalls, instead of forcefully reverting execution when an error occurs, we can design our functions to **return an explicit success or failure value**.The simplest approach is to return a **boolean**:

* true indicates that the function completed successfully
* false indicates that the function encountered an error

However, a boolean only tells us _that_ something went wrong—not _why_. To address this, Rust provides the Result type, which allows functions to return detailed error information.

### **Boolean-Based Error Handling**

To understand this idea more concretely, let’s consider a storage variable called EVEN\_ONLY, which is meant to store only even numbers.

```rust
use std::cell::RefCell;

thread_local!{
		static EVEN_ONLY : RefCell<u64> = RefCell::new(2);
}
```

To enforce this rule, we’ll add an a `set_even_only()` function that only allows even numbers to be saved to `EVEN_ONLY`.

```rust
use std::cell::RefCell;

thread_local!{
		static EVEN_ONLY : RefCell<u64> = RefCell::new(2);
}

#[ic_cdk::update]
fn set_even_only(new_number : u64) -> bool {
    if new_number % 2 == 0 {
		    // store the number
		    EVEN_ONLY.with_borrow_mut(|cell| *cell = new_number);
		    true
		} else {
				// do nothing
				false
		}
}
```

The `if` statement is used to check whether the number is even— if it is, store the number, if not, do nothing. Then, to return a success or failure indicator, we can return a boolean.

* `true` means that the function logic passed,
* `false` means that some business logic was violated or some other errors.

The drawback of this approach is that the function can only return a single false value, without indicating **why** the operation failed. If multiple error conditions are possible, there’s no way to tell which one occurred.To illustrate this limitation, we’ll update set\_even\_only to enforce two separate conditions:

1. The number has to be even, **and**
2. Only the owner can make changes

In the canister below, we added an `OWNER` variable, a constructor for `OWNER`, and applied an only owner access control to `set_even_only()`.

```rust
use std::cell::RefCell;

use candid::Principal;
use ic_cdk::api::msg_caller;

thread_local! {
        static EVEN_ONLY : RefCell<u64> = RefCell::new(2);

        // new owner
        static OWNER : RefCell<Principal> = RefCell::new(Principal::anonymous());
}

// initialize Owner
#[ic_cdk::init()]
fn init(_owner: Principal) {
    OWNER.with_borrow_mut(|cell| *cell = _owner);
}

#[ic_cdk::update]
fn set_even_only(new_number: u64) -> bool {
    let canister_owner = OWNER.with_borrow(|cell| cell.clone());

    // return false if not owner
    if msg_caller() != canister_owner {
        return false;
    }

    if new_number % 2 == 0 {
        EVEN_ONLY.with_borrow_mut(|cell| *cell = new_number);
        true
    } else {
        false
    }
}

```

A simple `false` return gives no indication of whether the failure was due to invalid input (odd number) or insufficient access permission (caller is not the owner). By having the function return a `Result<T, E>` type, we can communicate explicitly why the operation failed.

## The Result Type: `Result<T,E>`

The Result type, `Result<T,E>`, allows functions to return a clear success or error signal without failing the function and provide an attached value to either signal :

* Success signal: `Ok(payload)`
* Error signal: `Err(payload)`

Continuing from the even number example above, we’ll have the function return a `Result<T,E>` type. `Result` returns wraps your return value in an `Ok()` or `Err()` wrapper. `Ok()` indicating success and `Err()` indicator failure.

```rust
use std::cell::RefCell;

thread_local!{
		static EVEN_ONLY : RefCell<u64> = RefCell::new(2);
}

#[ic_cdk::update]
fn set_even_only(new_number : u64) -> Result<String,String> {
    if new_number % 2 == 0 {
		    EVEN_ONLY.with_borrow_mut(|cell| *cell = new_number);
		    
		    Ok("Success".to_string())
		 else{
				Err("Not an Even Number".to_string())
		 }
}
```

The value that’s wrapped is defined in the `Result<T,E>`.

* `T` is the type to return in the `Ok()` case, left hand side, and
* `E` is the type to return in the `Err()` case, right hand side.

If its a success, it returns “success” wrapped in by `Ok()` and

![Screenshot 2025-10-01 at 15.34.32.png](../.gitbook/assets/Screenshot_2025-10-01_at_15.34.32.png)

in the fail case, “Not an Even Number” wrapped in an `Err()`

![Screenshot 2025-10-01 at 15.34.42.png](../.gitbook/assets/Screenshot_2025-10-01_at_15.34.42.png)

### Using `enums` for Structured Errors

Using enums, we can define a fixed set of possible error cases that our function might return. For example, instead of just returning `"Not an Even Number".to_string()`, we can define an enum that captures the possible errors in our function:

```rust
#[derive(Debug)]
enum SetEvenError {
    NotEven,
    NotOwner,
}
```

Now we can update our `set_even_only` function to return a `Result<T, SetEvenError>`. This way, the caller knows whether the error was due to the number not being even, or because the caller was not the owner.

```rust
use std::cell::RefCell;

thread_local! {
    static EVEN_ONLY: RefCell<u64> = RefCell::new(2);
    static OWNER: RefCell<String> = RefCell::new("alice".to_string());
}

#[ic_cdk::update]
fn set_even_only(new_number: u64, caller: String) -> Result<String, SetEvenError> {
    let owner = OWNER.with(|o| o.borrow().clone());
    if caller != owner {
        return Err(SetEvenError::NotOwner);
    }

    if new_number % 2 != 0 {
        return Err(SetEvenError::NotEven);
    }

    EVEN_ONLY.with_borrow_mut(|cell| *cell = new_number);
    Ok("Success".to_string())
}

```

Another advantage of enums is that they are easy to extend. If later we wanted to add a new rule, like requiring the number to be less than 100, we could just add a new `TooLarge` case to the `SetEvenError` enum and update the function accordingly.

This structured approach makes it much clearer to both developers and users why a function failed, and it ensures that all possible error cases are considered at compile time.

To summarize, returning an error indicator is encouraged because:

1. Reverting implicates an inconsistent state roll-back due to asynchronous execution.
2. Provides Good UI to the client to resolve errors quickly.
