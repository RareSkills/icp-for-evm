# Introduction to Canister Programming with Motoko or Rust

![canisterprogram.jpg](./Introduction%20to%20Canister%20Programming%20with%20Motoko%20or%20Rust/canisterprogram.jpg)

This tutorial is targeted towards EVM developers to quickly get up to speed with programming Canister smart contracts in the `Motoko` or `Rust` language . It translates the Solidity equivalent data structures and language syntax to their ICP counterparts where possible.

The article is divided into two sections: `Motoko` and `Rust`. Navigate to your canister programming language of choice:

- [Canister Programming with Motoko](https://www.notion.so/Introduction-to-Canister-Programming-with-Motoko-or-Rust-11a09cb3e96280f79fa2f04f1a30534e?pvs=21)
- [Canister Programming with Rust](https://www.notion.so/Introduction-to-Canister-Programming-with-Motoko-or-Rust-11a09cb3e96280f79fa2f04f1a30534e?pvs=21)

# Canister Programming with Motoko

## Developer Environment

We recommend following along with this tutorial by setting up either one of the developer environments: 

- `DFX`,
- [`Motoko Playground`](https://m7sm4-2iaaa-aaaab-qabra-cai.raw.ic0.app) or
- [`ICP.Ninja`](http://ICP.Ninja)

## Declaring an Actor Contract

In Solidity, contracts are defined using the `contract` keyword, which encapsulates state variables and functions declared within the scope of the `contract`.

```solidity
contract SimpleContract{
		uint256 SomeVariable;
		
		function somefunc()public pure returns(uint256){
			return 0;
		}
}
```

In Motoko, instead of using the keyword `contract`, they use the `actor` keyword. For example, the equivalent `Motoko` code to the Solidity code above is:

```jsx
actor SimpleContract {
    var SomeVariable : Nat = 0;
    
    public func somefunc() : Nat {
        return 0;
    }
}
```

Interestingly enough, there are a lot of observable similarities Solidity and Motoko. Both have a similar structure where variables and functions are encapsulated within an `actor`(Motoko) or `contract`(Solidity). Global variables persist in the storage across state changes. Callable functions can perform computations and modify the state. 

If you are wondering why they use the keyword actor instead of contract, read more about the [**actor model for canisters**](https://internetcomputer.org/docs/current/motoko/main/writing-motoko/actors-async).

## Variables and Types

`Motoko` and `Solidity` variables behave similarly, they store their state in the storage and their value can be changed through functions.

In Solidity, this is how `Mutable` and `Immutable` variables are declared:

```solidity
contract MutableImmutableVariables{
	// Mutable Variable
	uint256 mutableNumber;
	
	// Immutable Variables
	uint immutable immutableNumber;
	uint256 constant constantNumber;
}
```

In Motoko, to declare `mutable` and `immutable variables`, it uses the keyword `var` and `let` respectively,

```jsx
actor SimpleContract {
		// Mutable Variable
    var mutableVariable : Nat = 0;
    
    // Immutable Variable
    let immutableVariable : Nat = 0;
}
```

followed by the `variable_name`, `type`, and an `initial value` (initialization is required, unlike solidity).

The main difference is primarily in the structure for declaring variables and the initialization requirement. 

To typecast a variable into a specific type, you’ll need to use the semicolon operator, `:` after the variable name. Here are some of the types you can use

- Text
- Bool
- Float
- Principal (Equivalent to address)
- Nat
- [and more](https://internetcomputer.org/docs/current/motoko/main/reference/language-manual#primitive-types)

```jsx
actor SimpleContract {
		// Mutable Variable
    var number : Nat = 0;
    var name : Text = 0;
    var isTrue: Bool = true;
    var decimals: Float = 0.1;
    var owner : principal = Principal.fromText("aaaaa-aa");
}
```

To assign value to a mutable variable within a function, use the `:=` operator. 

```jsx
actor SimpleContract {
		// Mutable Variable
    var x : Nat = 0;
    
    public func setX(x: Nat){
	    mutableVariable := x;
    }
}
```

Variables are private by default in Motoko. In Solidity, you can specify the visibility of the variable such as public, pr , but in Motoko, it i, therefore, to return the value of a variable, you need a function that specifically returns its value:

```solidity
actor Test {
  var x : Nat = 0; // private by default

  // Public function to set the value
  public func setX(newX : Nat) : () {
    x := newX;
  };

  // Public query function to access the value of x
  public query func getX() : async Nat {
    return x;
  }
}
```

We’ll discuss more about how to write functions in detail in the next section.

## Motoko Functions

Motoko functions are invokable units of code designed to execute deterministically, carrying out specific tasks or operations. To quickly grasp the programming concept, we can abstractly think that the way functions behave is similar to solidity functions.

### Passing Parameters

In Solidity, function arguments as well as their return values have to be explicitly declared and type casted.

```solidity
contract Solidity{
	
	function returnDouble(uint256 _x) public pure returns(uint256){
		return x*2;
	}

}
```

Functions in Motoko do too. The function below takes a Natural number (`_x : Nat`) as the function argument, and returns a Natural number `Nat`.

```solidity
actor Motoko{
	var x: Nat = 200;
	
	public setX(_x: Nat) : async Nat{
		x = _x;
		return x;
	}
}
```

Declaring function arguments is straight forward, the syntax is `variable_name : type`. The value passed to the function argument will be kept at the function call stack and can be used within the local scope of the function call. 

For two or more arguments, simply add another `variable_name : type` followed by a comma.

```solidity
public setX(_x: Nat, _y:Nat) : async Nat{
	x = _x;
	return x;
}
```

### Return Values

Return values need to have their type declared as well. If the function returns nothing, e.g, they only execute an operation, then it does not need a return type such as the `incrementX()` function below:

```solidity
actor NoReturnValue{
	var x : Nat = 0;

	public func incrementX(){
		x := x + 1;
	};
}
```

or in another method, we’ll use a semi colon followed by an empty tuple `: ()`.

```solidity
actor NoReturnValue{
	var x : Nat = 0;

	public func incrementX() : (){
		x := x + 1;
	};
}
```

An empty tuple indicates that it returns nothing.

To return a Natural(`Nat`) number, specify the return type as `: async Nat`. A semi colon `:` followed by the `async` keyword and the return type (`Nat`).

```solidity
actor NoReturnValue{
	var x : Nat = 0;

	public func incrementX() {
		x := x + 1;
	};

  public query func readX(): async Nat {
    return x;
  }
}
```

To return a `Text` or a `Bool`, use `async Text` and `async Bool`. 

```solidity
actor ReturnValues{

		public retText() : async Text{
			return "Hello World!";
		};

		public retBool() : async Bool{
			return true;
		}
																	
}
```

We can also return two or more values by using a tuple, which groups multiple values into a single return type, such as `(Nat, Nat)`.

```solidity
actor retTwoVal{
		public returnTwoValues() : async (Nat, Nat){
			return (24, 42);
		};
}
```

Canisters follow the `actor` model, what this means is that a canister communicates with external parties or vice versa asynchronously. Therefore, the return value would be specified to be an asynchronous return value. Hence why we must use the `async` keyword for `public` functions.  We’ll discuss this in more detail later at the Asynchronous article.

### Visibility Modifiers

In Solidity, there are a couple of visibility modifiers, namely `public`, `external`, `private` and `internal`. It dictates who can call the function. In Motoko, there are only two, which are `public` and `private`.

All the functions we have previously shown utilized the `public` keyword, it exposes the function externally, allowing any user or canister to call the function. 

```solidity
actor MotokoContract {
  var x : Nat = 0;

  public func setX(newX : Nat) : () {
    x := newX;
  };
  
  public query func getX(): async Nat{
	  return x;
  }
  
}
```

Anyone can call the `setX()` and `getX()` function.  And the return value for public functions must be asynchronous, hence using the `async` keyword.

To create a function that is **not** exposed publicly for internal logic purposes, we need to use the `private` keyword. The `updateX()` function below demonstrates this.

```solidity
actor MotokoContract {
  stable var x : Nat = 0;
  
  // Only functions within this contract can call this function
  private func updateX(newX : Nat){
     x := newX;
  };

  public func setX(newX : Nat) : () {
    updateX(newX);
  };

  public query func getX() : async Nat {
    return x;
  }
}

```

`updateX()` can only be called indirectly by another function within the contract, in this case, through the `setX()` function.

As a general syntax rule , the `visibility modifier` of a Motoko function is always declared first followed by the `func` keyword and the `function name`.

### Query and Update Calls

Motoko functions can be classified into two groups, `Update Calls` or `Query Calls`.

`Update calls` allows for **read and write** (state-changing) operations while `Query calls` allow for **read-only** operations.

```solidity
actor Test {
  var x : Nat = 0; // private by default

  // Update Call
  public func setX(newX : Nat) : () {
    x := newX;
  };

  // Query Call
  public query func getX() : async Nat {
    return x;
  }
}
```

`Update calls` do not need a specific modifier keyword to indicate it, but `Query calls` do. The `query` modifier keyword enforces the compiler to only perform read-only operations. 

```solidity
// Query calls need the query keyword modifier
public query func getX() : async Nat {
  return x;
}
```

Interestingly enough, if you perform a state-changing operation within a `Query Call`, it will temporarily change the variable state, but the change does not persist after the function call. To demonstrate what we mean by this, test out the actor canister below. Call `returnX()` , then call `getX()`. You can verify that the state-change does not persist after the query function call.

```solidity
actor QueryCall behavior {
  var x: Nat = 0;
  
  public query func returnX() : async Nat {
    x := 21;
    return x;
  };

  public query func getX() : async Nat {
    return x;
  }
}

```

## Solidity Data Structure Equivalents in Motoko

### Mapping

Solidity `mapping` data type creates a key-value store.

```solidity
// Solidity
mapping(address => uint) balances;

```

Motoko uses the `HashMap` module for similar key-value storage:

```jsx
import HashMap "mo:base/HashMap";
import Principal "mo:base/Principal";

actor BalancesManager {
    // Initialize the HashMap to store balances
    let balances = HashMap.HashMap<Principal, Nat>(5, Principal.equal, Principal.hash);

    // Function to add or update a balance for a given Principal
    public func addBalance(user: Principal, amount: Nat) : async () {
        let currentBalance = HashMap.get(balances, user) ?: 0; // Get current balance or default to 0
        HashMap.put(balances, user, currentBalance + amount); // Update the balance
    }

    // Function to remove a balance for a given Principal
    public func removeBalance(user: Principal) : async ?Nat {
        switch (HashMap.get(balances, user)) {
            case null {
                return null; // Return null if the user does not exist
            };
            case (?balance) {
                HashMap.remove(balances, user); // Remove the user from the map
                return balance; // Return the removed balance
            };
        }
    }

    // Function to retrieve a balance for a given Principal
    public func getBalance(user: Principal) : async ?Nat {
        return HashMap.get(balances, user); // Return the balance or null if not found
    }
}

```

To support address types directly, you would typically use `Text` or `Principal` as keys.

### Arrays

In Solidity, arrays can be fixed-size or dynamic:

```solidity
// Solidity
uint[] dynamicArray;
uint[5] fixedArray;

```

In Motoko, arrays are always dynamic, but you can limit the size by creating helper functions to enforce size constraints.

```jsx
actor ArrayManager {
    // Dynamic array example
    let dynamicArray : [Nat] = [1, 2, 3];

    // Fixed-size array example
    let fixedArray : [Nat] = Array.init<Nat>(5, 0); // Fixed to size 5 with initial value 0
}

```

### Structs

In Solidity, structs are a custom data type that can hold multiple data fields:

```solidity
// Solidity
struct User {
    string name;
    uint age;
}

User user = User("Alice", 25);

```

In Motoko, you use `object` types to define similar structures:

```jsx
actor UserManager {
    // Define a User type
    type User = {
        name : Text;
        age : Nat;
    };

    // Create a user instance
    let user : User = {
        name = "Alice";
        age = 25;
    };
}

```

## Constructors

In Solidity, you define a contract that functions similarly to an actor class. The contract has a special `constructor` that is executed once when the contract is deployed:

```solidity
// Solidity
pragma solidity ^0.8.0;

contract MyContract {
    uint public value;

    // Constructor
    constructor(uint _value) {
        value = _value; // Initialize value
    }
}

```

Here, `_value` is passed during deployment, setting the initial state of `value`.

In Motoko, an actor class does not have a dedicated `constructor` function like in Solidity. Instead, you define the actor class with parameters that are used for initialization directly in the actor body. This runs once when the actor is created.

```jsx
actor class MyActor(value : Nat) {
    var storedValue : Nat = value; // Initialize with passed value

    public func getValue() : async Nat {
        return storedValue;
    };
}

```

In this example, `value` is passed when the actor is instantiated, and `storedValue` is initialized with it.

## Require Statement

In Motoko, `assert` is equivalent to Solidity's `require` but without a custom error message.

```solidity
// Solidity
pragma solidity ^0.8.0;

contract NumberContract {
    uint public threshold = 100;

    function checkNumber(uint number) public pure {
        require(number <= threshold, "Number exceeds the threshold");
    }
}

```

- **If `number <= threshold`**: Execution continues.
- **If `number > threshold`**: Reverts with `"Number exceeds the threshold"`.

In motoko, the equivalent would be using the assert keyword.

```jsx
actor NumberActor {
    let threshold : Nat = 100;

    public func checkNumber(number : Nat) : async () {
        assert(number <= threshold); // Traps if number > threshold
    };
}
```

- **If `number <= threshold`**: Returns `()`.
- **If `number > threshold`**: Traps without an error message.

For a custom error message in Motoko, use:

```jsx
actor NumberActor {
    let threshold : Nat = 100;

    public func checkNumber(number : Nat) : async () {
        if (number > threshold) {
            throw Error.reject("Number exceeds the threshold");
        };
    };
}

```

- **If `number <= threshold`**: Execution continues.
- **If `number > threshold`**: Traps with `"Number exceeds the threshold"`.

## Accessing Global system Variables: Time

In Solidity, global system variables like `block.number` and `block.timestamp` provide information about the current block and its timestamp. However, in Motoko and the Internet Computer (ICP), there are no direct equivalents to Ethereum's `block.number` and `block.timestamp`, because ICP doesn't have a concept of blocks like a traditional blockchain.

Instead:

- `Motoko/ICP`: Access to time can be obtained using the `Time.now()` function, but there is no direct concept of block numbers.

- `Time.now()`: Returns the current timestamp in nanoseconds since the Unix epoch, useful for checking or logging time within canister functions.

Here’s an example of how to use `Time.now()` in Motoko

### Motoko (Using `Time.now()`)

```jsx
import Time "mo:base/Time";

actor TimeActor {
    var lastUpdated : Time = Time.now();

    public func update() : async () {
        lastUpdated := Time.now(); // Sets lastUpdated to the current timestamp
    };

    public func getLastUpdated() : async Time {
        return lastUpdated;
    }

    public func addTime(seconds: Nat) : async () {
        // Convert seconds to nanoseconds and add to lastUpdated
        lastUpdated := lastUpdated + seconds * 1_000_000_000; // 1 second = 1_000_000_000 nanoseconds
    }
}

```

The `lastUpdated` variable in the `TimeActor` stores the last updated timestamp in nanoseconds since the Unix epoch. 

The `update()` function sets `lastUpdated` to the current timestamp using `Time.now()`, while the `getLastUpdated()` function returns the current value of `lastUpdated`. 

Additionally, the `addTime(seconds: Nat)` function takes an input in seconds and adds that amount of time to `lastUpdated`. This conversion from seconds to nanoseconds is accomplished by multiplying the input by `1_000_000_000`. Therefore, if you call `addTime(30)` on this actor, it will add 30 seconds, equivalent to `30_000_000_000` nanoseconds, to the `lastUpdated` timestamp.

## Stable and Flexible Variables

**Stable variables** persist across canister upgrades and restarts, defined using the `stable` keyword. They are essential for maintaining critical state information. In contrast, **flexible variables** exist in volatile memory and are reset during upgrades, making them ideal for temporary data.

To declare a flexible or stable variable, by default, variables are flexible, we simply need to add the stable keyword to make the variable a stable variable like such:

```jsx
actor FlexibleExample {
    var flexibleCount: Nat = 0;
    stable var stableCount: Nat = 0;

    public func increment() : async () {
        flexibleCount := flexibleCount + 1;
    }

    public func getCount() : async Nat {
        return flexibleCount;
    }
    
    
    public func incrementStable() : async () {
        stableCount := stableCount + 1;
    }

    public func getCountStable() : async Nat {
        return stableCount;
    }
}
```

After incrementing `stableCount` to `3`, an update adding a reset function does not erase the previous value:

Calling `getCount()` after the update still returns `3`, demonstrating that stable variables retain their values, while flexible variables do not. 

# Canister Programming with Rust

On the Internet Computer (IC), you can write canisters in Rust by leveraging a few core libraries and features. Just like Solidity uses `contract` constructs and Motoko uses `actor` blocks, Rust canisters rely on a combination of **thread-local storage** and **attributes** (`#[ic_cdk::update]` or `#[ic_cdk::query]`) to define exposed "entry points" for your canister.

## Developer Environment

To follow along, you'll need:

- **DFX**
    
    Install the DFX tool from the [DFINITY SDK](https://smartcontracts.org/docs/developers-guide/install-upgrade-remove.html) to manage your canisters locally or on the IC.
    
- **IC SDK for Rust**
    
    This typically involves setting up your `Cargo.toml` with the right dependencies and configuring your `dfx.json` to recognize Rust-based canisters.
    

With these tools configured, you can compile and deploy your Rust canisters to the Internet Computer.

## Declaring a Canister Contract

In Solidity, you define contracts with the `contract` keyword. Here's a simple example:

```solidity
contract SimpleContract {
    uint256 SomeVariable;

    function somefunc() public pure returns(uint256) {
        return 0;
    }
}

```

A Rust-based canister, in contrast, manages data through **thread-local storage** instead of storing variables within a contract scope. The code below shows how we define a global state `SOME_VARIABLE` and a function `some_func()` that returns a static value:

```rust
use candid::{Nat, Principal};
use ic_cdk::storage;
use std::cell::RefCell;

thread_local! {
    static SOME_VARIABLE: RefCell<Nat> = RefCell::new(Nat::from(0 as u64));
}

#[ic_cdk::update]
fn some_func() -> Nat {
    Nat::from(0 as u64)
}

```

1. **`thread_local! { ... }`** – Creates a thread-local variable to hold persistent canister state.
2. **`RefCell<Nat>`** – Rust's interior mutability pattern, allowing us to mutate data behind an immutable reference.
3. **`#[ic_cdk::update]`** – Marks `some_func()` as an *update method*, meaning it can be called externally to modify or interact with state on the canister.

## Variables and Types

### Solidity Reference

```solidity
contract MutableImmutableVariables {
    // Mutable Variable
    uint256 mutableNumber;

    // Immutable Variables
    uint immutable immutableNumber;
    uint256 constant constantNumber;
}

```

Here, `mutableNumber` can be changed throughout the contract's lifecycle, while `immutable` and `constant` variables are set at compile time or construction time and cannot be changed.

### Rust Thread-Local Storage

In Rust canisters, **all** state is typically stored in "thread-local" blocks. We can mimic "mutable" vs. "immutable" variables by storing them in `RefCell` (mutable) or as constants (immutable). Below is an example:

```rust
use candid::{Nat, Principal};
use std::cell::RefCell;

thread_local! {
    // Mutable Variable
    static MUTABLE_NUMBER: RefCell<Nat> = RefCell::new(Nat::from(0 as u64));

    // Immutable Variables (through constants)
    static IMMUTABLE_NUMBER: Nat = Nat::from(42 as u64);
}

// Constants can be declared outside thread_local
const CONSTANT_NUMBER: u64 = 100;

```

- **`MUTABLE_NUMBER`** uses a `RefCell` for interior mutability—this value can be changed over time via an `update` function.
- **`IMMUTABLE_NUMBER`** is assigned once in the `thread_local` block and is not modified afterward.
- **`CONSTANT_NUMBER`** is a plain Rust constant, effectively a compile-time constant.

### Common Rust Types for Canisters

- **`Nat`** (from the `candid` crate) – Arbitrary-precision natural number (like Solidity's `uint256`).
- **`Principal`** – Acts similarly to Ethereum `address`.
- **`String`** – Equivalent to `string` in Solidity, or `Text` in Motoko.
- **`bool`** – Boolean type.
- **`f64`** – Floating-point numbers (though be cautious using floats in financial logic).

### Accessing and Modifying State

Below is a pair of "setter" and "getter" functions:

```rust
#[ic_cdk::update]
fn set_number(new_value: Nat) {
    MUTABLE_NUMBER.with(|number| {
        *number.borrow_mut() = new_value;
    });
}

#[ic_cdk::query]
fn get_number() -> Nat {
    MUTABLE_NUMBER.with(|number| {
        number.borrow().clone()
    })
}

```

- **`#[ic_cdk::update]`** – Marks an endpoint that can **change** canister state.
- **`#[ic_cdk::query]`** – Marks a **read-only** function; these calls typically do not alter canister state.

Inside each function, we invoke `MUTABLE_NUMBER.with(...)` to access the `RefCell`. If we just need to **read**, we call `borrow()`. If we want to **write**, we call `borrow_mut()`.

## Rust Canister Functions

Rust canister functions declare their intent using attributes:

1. **`#[ic_cdk::update]`** – Allows the function to change state; known as an *update call* on the IC.
2. **`#[ic_cdk::query]`** – Read-only calls that do not persist state changes.

### Return Values

Rust canister functions can return any type that implements the appropriate Candid traits. For single values:

```rust
// Return single value
#[ic_cdk::query]
fn get_text() -> String {
    "Hello World!".to_string()
}

```

If multiple values are needed, you can return a tuple:

```rust
// Return multiple values using tuple
#[ic_cdk::query]
fn get_two_values() -> (Nat, Nat) {
    (Nat::from(24 as u64), Nat::from(42 as u64))
}

```

This is roughly analogous to Solidity's ability to return multiple values from a single function, or Motoko's tuple returns.

### Visibility and Access Control

In Solidity, you have `public`, `private`, `internal`, etc. In Rust canisters, a function is **externally callable** only if it has an `#[ic_cdk::update]` or `#[ic_cdk::query]` attribute. Other functions remain private to your canister code:

```rust
// Public function (can be called externally)
#[ic_cdk::update]
fn public_function(value: Nat) {
    private_function(value);
}

// Private function (internal use only)
fn private_function(value: Nat) {
    // Implementation
}

```

Since the private function has no IC attribute, it cannot be invoked externally via a canister call.

## Common Data Structures

### Using HashMaps in Rust as the Solidity's equivalent to mapping

Solidity's `mapping(address => uint)` is typically replaced by Rust's `HashMap<Principal, Nat>`. Here's a simple example for a balance-tracking system:

```rust
use std::collections::HashMap;
use candid::{Nat, Principal};

thread_local! {
    static BALANCES: RefCell<HashMap<Principal, Nat>> = RefCell::new(HashMap::new());
}

#[ic_cdk::update]
fn add_balance(user: Principal, amount: Nat) {
    BALANCES.with(|balances| {
        let mut balances = balances.borrow_mut();
        let current = balances.get(&user).cloned().unwrap_or_else(|| Nat::from(0 as u64));
        balances.insert(user, current + amount);
    });
}

#[ic_cdk::query]
fn get_balance(user: Principal) -> Option<Nat> {
    BALANCES.with(|balances| {
        balances.borrow().get(&user).cloned()
    })
}

```

- **`BALANCES`**: Thread-local `HashMap`.
- **`add_balance`**: Looks up the user's current balance, defaults to 0 if not found, and then writes an updated balance back to the map.
- **`get_balance`**: Returns an `Option<Nat>` which is `None` if the user has no entry.

### Structs (Similar to Solidity's Structs)

In Solidity, you might define a struct like this:

```solidity
struct User {
    string name;
    uint age;
}

```

In Rust, you can define a `struct` and store it in a `HashMap` as shown below:

```rust
#[derive(candid::CandidType, Clone)]
struct User {
    name: String,
    age: Nat,
}

thread_local! {
    static USERS: RefCell<HashMap<Principal, User>> = RefCell::new(HashMap::new());
}

#[ic_cdk::update]
fn add_user(user: User) {
    USERS.with(|users| {
        users.borrow_mut().insert(ic_cdk::caller(), user);
    });
}

```

- **`#[derive(candid::CandidType]`** – Makes the struct serializable with Candid, the IDL (Interface Definition Language) used by the IC.
- **`ic_cdk::caller()`** – Retrieves the principal of the caller, which can serve as the key in the user map.

## Initialization (Constructor Equivalent)

In Solidity, a `constructor` is executed once during deployment. Rust canisters instead define an **`#[ic_cdk::init]`** function, which runs exactly once on canister install:

```rust
#[ic_cdk::init]
fn init() {
    let initial_value = Nat::from(100 as u64);
    SOME_VALUE.with(|val| {
        *val.borrow_mut() = initial_value;
    });
}

```

- **`#[ic_cdk::init]`** is automatically invoked by the IC upon canister creation.
- You can set up your initial state, similar to a Solidity constructor.

## Error Handling and Assertions

### Using `Result`

Rust canisters often use the `Result<T, E>` pattern for functions that may fail:

```rust
#[ic_cdk::update]
fn check_number(number: Nat) -> Result<(), String> {
    let threshold = Nat::from(100 as u64);
    if number > threshold {
        return Err("Number exceeds threshold".to_string());
    }
    Ok(())
}

```

- If `number > threshold`, we return an `Err(...)`.
- Otherwise, the function returns `Ok(())`.

### Using `ic_cdk::trap` (Similar to Solidity's `require`)

If you want the canister to trap immediately (like `require(false)` in Solidity):

```rust
// Using trap (similar to Solidity's require)
#[ic_cdk::update]
fn require_condition(value: Nat) {
    if value == Nat::from(0 as u64) {
        ic_cdk::trap("Value cannot be zero");
    }
}

```

- This triggers a **trap**, rolling back the call and providing an error message.
- In Solidity terms, it's similar to a `require` that reverts on failure.

## Time Management

In Solidity, `block.timestamp` can be accessed. In Rust canisters on the IC, you retrieve the current timestamp (in nanoseconds) via `ic_cdk::api::time()`:

```rust
use ic_cdk::api::time;

#[ic_cdk::query]
fn get_current_time() -> u64 {
    time()  // Returns nanoseconds since 1970-01-01
}

thread_local! {
    static LAST_UPDATED: RefCell<u64> = RefCell::new(0);
}

#[ic_cdk::update]
fn update_timestamp() {
    LAST_UPDATED.with(|t| {
        *t.borrow_mut() = time();
    });
}

```

1. **`time()`** – Returns a `u64` representing the number of nanoseconds from the Unix epoch.
2. **`LAST_UPDATED`** – Stores the last time someone called `update_timestamp()`.

## Conclusion

1. **State**: Define your canister state using `thread_local!` for data you want to preserve between calls, or stable structures for data you want to preserve between *upgrades*.
2. **Entry Points**: Annotate functions with `#[ic_cdk::update]` or `#[ic_cdk::query]` to expose them externally.
3. **Initialization**: Use `#[ic_cdk::init]` for constructor-like setup logic.

Below is a quick reference for dependencies in your `Cargo.toml`:

```toml
[dependencies]
candid = "0.8"
ic-cdk = "0.7"
ic-stable-structures = "0.5"
serde = { version = "1.0", features = ["derive"] }

```

This ensures you have all the libraries needed (Candid, IC CDK, stable structures, and Serde) for canister development.