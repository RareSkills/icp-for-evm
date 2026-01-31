# Asynchronous Execution

In Ethereum, all transactions execute atomically. Transactions that are atomic either commits state-changes or non at all. This makes it easy for the developers to implement circuit breakers in the transaction is invalid for any reason. On the contrary, ICP transactions execute both atomically and asynchronously. Asynchronous execution would only partially roll back state-changes. In this article, we’ll look at when a canister function executes atomically or asynchronously.

## Execution is Atomic If It Stays Within The Canister

Canister functions execute atomically, but only if the execution stays **within the same canister** and it make a call any other asynchronous system APIs such as the `Call` system API.

In an atomic function or execution, state changes are committed **last**, only if the function completes successfully. Conceptually, it looks like this:

```
							   (provisional)                    (finalized)
  |----------    state changes     -----------|   commit state
  ^                                           ^       
start                                        end    
```

During execution, all state changes are **provisional** (temporary). They are only finalized and committed to the blockchain when the function finishes without panicking or trapping.

To understand this behavior more concretely, consider the `double_increment()` function below, which executes atomically because all of its execution stays within the same canister and no inter-canister calls are made.

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

 

Now consider a version of `double_increment()` that always reverts using `assert!`. Because the function panics before completing, none of its state changes are finalized, and `X` remains `0`.

```rust
use std::cell::RefCell;

thread-local!{
		static X: RefCell<u64> = RefCell::new(0);
}

fn increment() {
		X.with_borrow_mut(|cell| *cell += 1);
}

fn double_increment() -> bool {
		increment();
		increment();
		
		// *** Force Revert ***
		assert!(false);
		
		true
}
```

This confirms that atomic execution guarantees **all-or-nothing** state changes, but only as long as execution remains entirely within a single canister. This behavior does **not** apply once a function performs an inter-canister call.

### Functions That Perform Inter-Canister Calls Are Not Atomic

When a function performs an inter-canister call, its execution becomes asynchronous.

In an asynchronous execution, state changes are no longer committed only at the end of the function. Instead, they may be finalized at specific points during execution, such as before and after an inter-canister call.

To understand how this works in practice, consider `Canister-A` (the caller) and `Canister-B` (the callee) shown below.

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

    result.is_ok()
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

ic_cdk**::export_candid!**();
```

The `foo()` function in `Canister-A` first updates its local state by incrementing `X`. At this point, execution has not yet crossed a canister boundary, so this update is part of the first execution segment.

When `foo()` makes an inter-canister call to `B.bar()`, execution is suspended and control is returned to the system. Before this call is issued, all state changes made so far in Canister A are committed. As a result, the increment to `X` becomes permanent.

Canister B then executes the `bar()` function in a separate execution context. If `bar()` completes successfully, any state changes it makes are committed independently in Canister B.

Once the inter-canister call returns, execution in Canister A resumes from the `await`. Even if `foo()` panics or traps after this point, the state changes committed before the inter-canister call—and any state changes committed in Canister B—are not rolled back.

To observe this behavior in practice, deploy `Canister-A` and `Canister-B` and follow the steps below.

1. Call `foo()` on Canister A.
2. Then call `get_x()`.

Even though `foo()` always panics after the inter-canister call, `get_x()` returns **1**, showing that the increment to `X` was **not** rolled back.

![Screenshot 2025-11-25 at 19.10.34.png](asynchronous-execution/screenshot-2025-11-25-at-19.10.34.png)

Atomic execution guarantees an `all-or-nothing` outcome: either all state changes are committed, or none of them are. In this example, however, the state change to `X` survived even though `foo()` panicked at the end of its execution. This shows that `foo()` does not execute atomically, because it performs an inter-canister call.

In the next section, we’ll discuss how and when a canister function executes asynchronously.

## **Functions That Perform Inter-Canister Calls Execute Asynchronously**

When a function performs an inter-canister call, its execution becomes asynchronous. The reason is that the inter-canister call System API, call, itself responds asynchronously. As a result, any function that uses this API cannot complete its execution in a single atomic step.

In this section, we’ll focus on how state changes are committed in the presence of asynchronous execution. Although our examples use inter-canister calls, the same execution model applies to other asynchronous System APIs, such as HTTP outcalls, which we’ll cover later.

### State Changes are Committed Incrementally

In an atomic function, state changes are committed **only once**, at the very end of the function call, provided the function completes successfully.

Asynchronous functions behave differently. When a function performs an inter-canister call, its execution is split into multiple segments, and state changes may be committed **incrementally** at specific checkpoints during execution.

For a function that performs a single inter-canister call, these commit points are:

- **Before the inter-canister call**, all state changes made so far in the caller canister are finalized and committed.
- **When the callee canister completes successfully,** the callee canister commits its state changes.
- W**hen the caller function finishes execution successfully**, any remaining state changes made after the inter-canister call are finalized and committed in the caller canister.

```
													(Inter-canister call)
			    													^           
Start |--- **state changes** --|-- **state changes** --|-- **state changes** --| End  
                           ^                   ^                   ^
                         Commit              Commit              Commit
                         Point               Point               Point
```

Rather than executing as one atomic transaction, the function can be understood as a sequence of three consecutive atomic executions. 

### State-changes are committed before the inter-canister call is made

The first commit point in an asynchronous function occurs **immediately before** an inter-canister call is issued. At this point, all state changes made so far in the caller canister are finalized and committed.

The example below highlights this behavior. The code is annotated to show where the first commit point occurs.

```rust
**use** std**::**cell**::**RefCell;
**use** ic_cdk**::**{call**::**Call, update};
**use** candid**::**Principal;

**thread_local!**{
		static X **:** RefCell<u64> **=** RefCell**::new**(0);
}

#[update]**async** **fn** **foo**(callee**:**Principal)**->** bool{
		// update x        
		X**.with_borrow_mut**(**|**cell**|** *****cell **+=** 1);
		
		// *** STATE IS COMMITTED ***
		**let** _result **=** Call**::unbounded_wait**(callee,"increment_y")**.await**;

		// update x
		X**.with_borrow_mut**(**|**cell**|** *****cell **+=** 2);                

		**return** true;
} 
ic_cdk**::export_candid!**();
```

Any state changes made **before the first await** are permanently committed to the canister’s state. As a result, if the function panics or traps **during** the inter-canister call—or at any point after it—the earlier change (x += 1) is **not** rolled back.

This behavior changes, however, if the function panics **before** reaching the inter-canister call. In that case, execution never reaches the first commit point, and all state changes from that segment are discarded.

The example below demonstrates this case:

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

So far, we’ve seen that the first commit point occurs **before** an inter-canister call is issued, when the caller canister commits all state changes made up to that point. The next commit point occurs when the inter-canister call itself completes successfully.

In the example shown below, after `A::foo()` commits its own state changes before issuing the inter-canister call, it invokes `B::bar()` using the `Call` system API. If `B::bar()` completes successfully, all state changes made during that execution are finalized and committed in canister B.

Canister `A`

```rust
use std::cell::RefCell;

use candid::Principal;
use ic_cdk::{call::Call, update};

thread_local! {
    static X: RefCell<u64> = RefCell::new(0);
}

#[update]
async fn foo(callee: Principal) -> bool {
    // update x
    X.with_borrow_mut(|cell| *cell += 1);

    // *** STATE IS COMMITTED ***

    let _result = Call::unbounded_wait(callee, "increment_y").await;

    // update x
    X.with_borrow_mut(|cell| *cell += 1);

    true
}

ic_cdk::export_candid!();
```

Canister `B`

```rust
use std::cell::RefCell;

thread_local! {
    static Y: RefCell<u64> = RefCell::new(0);
}

fn bar() -> bool {
    // update Y
    Y.with_borrow_mut(|cell| *cell += 1);
    true
}

ic_cdk::export_candid!();
```

If the inter-canister call itself panics or traps inside `B::bar()`, then **no** state changes are committed in canister `B`, and `A::foo()` continues executing from the `await` with an error result.

### Reverts after an inter-canister call returns will not roll back the previous state-changes

If `foo()` reverts **after** the inter-canister call, and `B::bar()` has already executed successfully, then:

- The state changes in `B` are **not** rolled back.
- The state changes made in `A` **before** the inter-canister call are also **not** rolled back.

```rust
use std::cell::RefCell;

use candid::Principal;
use ic_cdk::{call::Call, update};

thread_local! {
    static X: RefCell<u64> = RefCell::new(0);
}

#[update]
async fn foo(callee: Principal) -> bool {
    // update x
    X.with_borrow_mut(|cell| *cell += 1);

    // *** STATE IS COMMITTED ***

    let _result = Call::unbounded_wait(callee, "increment_y").await;

    // update x
    X.with_borrow_mut(|cell| *cell += 2);

    true
}

ic_cdk::export_candid!();
```

This pattern differs from the EVM since cross-contract calls are atomic, and if the main function reverts, then state-changes made to other contracts would be rolled back.

### The remaining state-changes are committed after the inter-canister call

So far, we’ve seen two commit points in an asynchronous execution:

1. The caller commits its state changes **before** issuing the inter-canister call, and
2. The callee commits its state changes **when the call completes successfully**.

The **third and final commit point** occurs when the caller’s asynchronous function finishes execution successfully. At this stage, any remaining state changes made **after** the await are finalized and committed in the caller canister.

The example below illustrates this final commit point:

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

![Screenshot 2025-11-06 at 11.38.57.png](asynchronous-execution/90286fce-dd62-4efa-83bd-b7992cf88acb.png)

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