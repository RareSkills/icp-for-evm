# Asynchronous Execution And It’s Effect On State-Changes

In this article, we’ll look at when a canister function executes atomically or asynchronously, as well as its affects on state-changes. 

## Execution is Atomic If It Stays Within The Canister

Canister functions execute atomically, but only as long as execution stays **within the same canister** and it does **not** perform any inter-canister calls or other asynchronous system API calls.

In an atomic function, state changes are committed **last**, only if the function completes successfully. Conceptually, it looks like this:

```
							   (provisional)                    (finalized)
  |----------    state changes     -----------|   commit state
  ^                                           ^       
start                                        end    
```

During execution, all state changes are **provisional** (temporary). They are only finalized and committed to the blockchain when the function finishes without panicking or trapping.

The `double_increment()` function below executes atomically, because everything happens inside the canister and no inter-canister calls are made.

```rust
use std::cell::RefCell;

thread-local!{
		static X : RefCell<u64> = RefCell::new(0);
}

fn increment()  {
		X.with_borrow_mut(|cell| cell += 1;);
}

fn double_increment()-> bool{
		increment()
		increment()
		return true;
}
```

If an atomic function panics or traps mid-execution, then **none** of its state changes are committed:

```
|----- state changes -----X------|   ✕ no state is committed
^                         ^      ^       
start                   panic   end    
```

 

Here is a version of `double_increment()` that always reverts using `assert!`. Because `double_increment()` always reverts, none of its state-changes are finalized, so `X` stays at `0`.

```rust
static x : u64 = 0;

fn increment()  {
		x += 1;
}

fn double_increment()-> bool{
		increment();
		increment();
		
		// *** Force Revert ***
		assert!();
		
		return true;
}
```

This atomic behavior applies only to execution that stays within the canister. It does **not** apply to functions that perform inter-canister calls.

### Functions That Perform Inter-Canister Calls Are Not Atomic

Functions that make an inter-canister call would not execute atomically. To prove this statement, we’ll demonstrate it with an example.

Within Canister `A` below, the function `foo()` increments variable `X` and then makes an inter-canister call to `B.bar()`. After the inter-canister call, it forcefully reverts with `assert!()`. **Observe that the state change that happens before the inter-canister call is not rolled back.**

Canister `A`:

```rust
use candid::Principal;
use ic_cdk::call::Call;
use std::cell::RefCell;

thread_local!{
		static X : RefCell<u64> = RefCell::new(0);
}

#[ic_cdk::update]
async fn foo(callee: Principal) -> bool {

		// update x
		X.with_borrow_mut(|cell| *cell += 1);
		
		// inter-canister call
    let result = Call::bounded_wait(callee, "bar").await;
        
    // *** Forced Panic *** 
    assert!(false, "always revert");

    result.is_ok();
}

#[ic_cdk::query]
async fn get_x() -> u64 {
	X.with_borrow(|cell| *cell)
}

ic_cdk::export_candid!();
```

Canister `B`:

```rust
#[ic_cdk::update]
fn bar(){
		// does nothing
}
```

Deploy canister `A` and `B`. 

1. Call `foo()`, then
2. Call `get_foo()`. `get_x()` would return **1**. 

Even though `foo()` always panics after the inter-canister call, the increment to `X` was **not** rolled back.

![Screenshot 2025-11-25 at 19.10.34.png](Asynchronous%20Execution%20And%20It%E2%80%99s%20Effect%20On%20State-Ch/Screenshot_2025-11-25_at_19.10.34.png)

Atomic functions should either commit **all** of their state changes or **none**. In this example, the state change survived even though `foo()` reverts at the end. This proved that `foo()`, a function that performs an inter-canister call, is **not** atomic.

In the next sections, we’ll discuss when and how a canister function executes asynchronously.

## Function Perform Inter-Canister Calls Execute Asynchronously

The inter-canister call System API, `call`, responds asynchronously, so any function that uses it will also execute asynchronously. We’ll explicitly focus on how state-changes are committed in context of an asynchronous function.

The asynchronous execution model described in this section applies not only to inter-canister calls, but also to other async System APIs such as HTTP outcalls (which we’ll cover later).

### State Changes are Committed Incrementally

Atomic functions commit state changes **only once**, at the end of the function call.

By contrast, async functions that perform inter-canister calls commit state changes **incrementally**, at certain checkpoints. For a single inter-canister call, the checkpoints are:

- **Before** the inter-canister call is made (state-changes are committed on caller canister).
- **When the callee canister returns successfully** (state-changes are committed on the callee canister).
- **When the caller function finally finishes**. (state-changes are committed on the caller canister again)

We can think of the entire function’s execution as three consecutive atomic transactions. Conceptually it looks like this:

```
													(Inter-canister call)
			    													^           
Start |--- **state changes** --|-- **state changes** --|-- **state changes** --| End  
                           ^                   ^                   ^
                         Commit              Commit              Commit
                         Point               Point               Point
```

### State-changes are committed before the inter-canister call is made

Before the inter-canister call is made, the caller’s state changes are committed. The code below is annotated to show where the first commit point happens.

```rust
static x : u64 = 0;

async fn foo()->bool{

		x += 1;
		
		// *** STATE IS COMMITTED ***
		Call::unbounded_wait("B","increment_y").await;
		
		x += 2;
		
		return true;
} 
```

Any state changes up to the first `await` are permanently committed in the canister’s state. Any panic or revert that happens **during** the inter-canister call (**or after**) will **not** undo the `X += 1;` change.

However, if the function panics **before** the inter-canister call, then those earlier changes are rolled back. For example:

```rust
use candid::Principal;
use ic_cdk::{call::Call, update};
use std::cell::RefCell;

thread_local!{
		static X : RefCell<u64> = RefCell::new(0);
}

#[update]
async fn foo()(callee: Principal) -> bool {

		// Rolled Back
		X.with_borrow_mut(|cell| *cell += 1);
		
		// Panic Before Inter-canister call
    assert!(false, "always revert");
    
    // STATE IS COMMITTED
    let result = Call::bounded_wait(callee, "bar").await;
    result.is_ok()        
}

ic_cdk::export_candid!();
```

Because the panic happens before the `await`, the whole first segment aborts and its state changes are not committed.

### State-changes are committed when the inter-canister call returns successfully

The first commit happens on the caller canister before the inter-canister call. The second commit happens on the **called canister**, but only if the inter-canister call executes successfully.

After `A::foo()` commits its own state changes (before the call), it invokes `B::bar()` using the `Call` System API. If `B::bar()` completes successfully, it commits its own state changes to canister `B`.

`A`

```rust
static x : u64 = 0;

async fn foo()->bool{
		x += 1;
		
		let result = Call::unbounded_wait("B","increment_y").await;
		
		x += 1;
		
		return true;
} 
```

`B`

```rust
static y : u64 = 0;

fn bar()->bool{
		y += 1;
		return true
		// STATE CHANGES ARE COMMITTED
} 
```

If the inter-canister call itself panics or traps inside `B::bar()`, then **no** state changes are committed in canister `B`, and `A::foo()` continues executing from the `await` with an error result.

### Reverts after an inter-canister call returns will not roll back the previous state-changes

If `foo()` reverts **after** the inter-canister call, and `B::bar()` has already executed successfully, then:

- The state changes in `B` are **not** rolled back.
- The state changes made in `A` **before** the inter-canister call are also **not** rolled back.

```rust
static x : u64 = 0;

async fn foo()->bool{
		x += 1;
		
		// (state changes are NOT reverted)
		let result = Call::unbounded_wait("B","increment_y").await;
		// (state changes at canister B are NOT reverted)
		
		// *** Forced Panic *** 
    assert!(false, "always revert");
		
		x += 1;
		
		return true;
} 
```

This pattern differs from the EVM since cross-contract calls are atomic, and if the main function reverts, then state-changes made to other contracts would be rolled back.

### The remaining state-changes are committed after the inter-canister call

The third and final commit point is when the caller’s async function finishes successfully. After:

1. The pre-call segment commits (caller A, before the call), and
2. The callee’s segment commits (canister B, inside `bar()`),

the remaining state changes in the caller (after the `await`) are committed if the function completes without panicking.

```rust
static x : u64 = 0;

async fn call_b()->bool{
		x+=1;
		// STATE IS COMMITTED
		
		Call::unbounded_wait("B","increment_y").await;
		// STATE AT CANISTER B IS COMMITTED
		
		x+=2;
		return true;
		// STATE IS COMMITED
} 
```

If we revert **after** `X += 2;`, like this

```rust
static x : u64 = 0;

async fn call_b()->bool{
		x+=1;
		// STATE IS COMMITTED
		
		Call::unbounded_wait("B","increment_y").await;
		// STATE AT CANISTER B IS COMMITTED
		
		x+=2;
		
		
		// *** Forced Panic *** 
    assert!(false, "always revert");
    
		return true;
		// STATE IS COMMITED
} 
```

Only the state changes from the **current** segment (`X += 2;`) are rolled back. The earlier commits:

- `X += 1;` (in canister A, before the call), and
- `Y += 1;` (in canister B, inside `bar()`)

remain permanently applied.

We can think of how an asynchronous function executes with one inter-canister call as three separate atomic executions that make up the entire transaction **M1**, **M2**, **M3**.

![Screenshot 2025-11-06 at 11.38.57.png](Asynchronous%20Execution%20And%20It%E2%80%99s%20Effect%20On%20State-Ch/90286fce-dd62-4efa-83bd-b7992cf88acb.png)

Each message executes atomically and commits its own state changes independently. Reverting an atomic segment would only revert its state-changes, but it would not revert the previous segments if any.

## Functions With Multiple Inter-Canister Calls

On each inter-canister call, the function splits its execution and adds two more atomic segments. If there were two inter-canister calls, then we would have 5 atomic segments.

```rust
static x : u64 = 0;

async fn call_b()->bool{
		x+=1;
		// STATE IS COMMITTED
		
		Call::unbounded_wait("B","increment_y").await;
		// STATE AT CANISTER B IS COMMITTED
		
		x+=2;
		
		Call::unbounded_wait("B","increment_y").await;
		// STATE AT CANISTER B IS COMMITTED
		
		return true;
		// STATE IS COMMITED
} 
```

- The first atomic execution is before the first inter-canister call.
- The second is at the first inter-canister call.
- The third is the codes in between the inter-canister calls.
- The fourth is the second inter-canister call and
- lastly, the codes after the second inter-canister call where the function finishes execution.

While waiting for the inter-canister call, the canister does not stall and block other transactions. It can process other transactions. We’ll discuss the non-blocking property of canisters in the next article.