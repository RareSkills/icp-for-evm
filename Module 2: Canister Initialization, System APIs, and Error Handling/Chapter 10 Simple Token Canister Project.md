# Simple Token Canister Project

In this tutorial, we'll build a simple token canister that implements the essential features of an ERC-20 token. The goal of this project is to tie together the previous lessons into a working simple token canister with the following core logic implementation:

**Token metadata:**

* `name()`
* `symbol()`

**Core token logic :**

* `mint()`
* `totalSupply()`
* `balanceOf()`
* `transfer()`

**Allowance / approval :**

* `allowance()`
* `approve()`
* `transferFrom()`

**Note**: On ICP, token interface standards are defined by the [ICRC specifications](https://www.notion.so/Simple-Token-Canister-Project-24a09cb3e96280f3b0bede0d7403d7f5?pvs=21) (Internet Computer Request for Comments). [**ICRC-1**](https://internetcomputer.org/docs/defi/token-standards/icrc-1) covers the core fungible token functions, and [**ICRC-2**](https://internetcomputer.org/docs/defi/token-standards/icrc-2) adds the approve / transferFrom pattern.

**Important:** Unlike Ethereum, canisters on ICP cannot emit on-chain events like `Transfer(...)`. There’s no built-in event log that wallets or indexers can subscribe to.

If you want event-like behavior—for example, a verifiable transfer history—you need to design and persist that history in the canister state and expose it via query methods. We won’t do that here. For reference, the token standard for exposing a transaction/block log in ICP is [ICRC-3](https://github.com/dfinity/ICRC-1/blob/main/standards/ICRC-3/README.md). It standardizes how clients fetch (and verify) a ledger’s transaction history

To kick start, create a new dfx project called `token_canister`.

```rust
dfx new token_canister --type rust --no-frontend
```

Clear `lib.rs` and insert the code examples into the file.

## The Token’s `Name` and `Symbol`

Besides the canister’s address, our token is identifiable through their name and symbol. Within the thread local storage, define the token’s `NAME` and `SYMBOL` string variable:

```rust
use std::cell::RefCell;

thread_local!{

		// **** NEW CODE HERE ****
		static NAME : RefCell<String> = RefCell::new(String::new()); 
		static SYMBOL : RefCell<String> = RefCell::new(String::new());
}

ic_cdk::export_candid!();
```

To make these variables queryable, add a query function for both the `NAME` and `SYMBOL`: `name()` and `symbol()` respectively.

```rust
use std::cell::RefCell;

thread_local! {
    static NAME : RefCell<String> = RefCell::new(String::new());
    static SYMBOL : RefCell<String> = RefCell::new(String::new());
}

// **** NEW CODE HERE ****
#[ic_cdk::query]
fn name() -> String {
    NAME.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn symbol() -> String {
    SYMBOL.with_borrow(|cell| cell.clone())
}

ic_cdk::export_candid!();
```

### The `init()` function

Both `NAME` and `SYMBOL`'s value will be initialized when the canister is first deployed. To handle this, define an `init()` function and mark it with the `#[ic_cdk::init]` attribute; this function will initialize `NAME`, `SYMBOL`, and any other state variables that are needed.

```rust
use std::cell::RefCell;

thread_local! {
		static NAME : RefCell<String> = RefCell::new(String::new());
		static SYMBOL : RefCell<String> = RefCell::new(String::new());
}

// **** NEW CODE HERE ****
#[ic_cdk::init]
fn init(name: String, symbol: String) {
    NAME.with_borrow_mut(|cell| *cell = name);
    SYMBOL.with_borrow_mut(|cell| *cell = symbol);
}

#[ic_cdk::query]
fn name() -> String {
    NAME.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn symbol() -> String {
    SYMBOL.with_borrow(|cell| cell.clone())
}

ic_cdk::export_candid!();
```

## Token Balance and Accounting: `Balances` and `TOKEN_SUPPLY`

To account for the amount of tokens each address owns, we will track them through a `HashMap` data structure. It will map a `Principal` type (address of a canister or user) to a `u128` number (amount of tokens they own):

```rust
static BALANCES : RefCell<HashMap<Principal, u128>> = RefCell::new(HashMap::new());
```

Insert `BALANCES` into the thread-local storage:

```rust
use std::{cell::RefCell, collections::HashMap};
use candid::Principal;

thread_local! {
        static NAME : RefCell<String> = RefCell::new(String::new());
        static SYMBOL : RefCell<String> = RefCell::new(String::new());
        
        // **** NEW CODE HERE ****
        static BALANCES : RefCell<HashMap<Principal, u128>> = RefCell::new(HashMap::new());

}

#[ic_cdk::init]
fn init(name: String, symbol: String) {
    NAME.with_borrow_mut(|cell| *cell = name);
    SYMBOL.with_borrow_mut(|cell| *cell = symbol);
}

#[ic_cdk::query]
fn name() -> String {
    NAME.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn symbol() -> String {
    SYMBOL.with_borrow(|cell| cell.clone())
}

ic_cdk::export_candid!();
```

Create a query function to retrieve the token balance for a given principal. We’ll name it `balance_of()`:

```rust
use std::{cell::RefCell, collections::HashMap};

use candid::Principal;

thread_local! {
        static NAME : RefCell<String> = RefCell::new(String::new());
        static SYMBOL : RefCell<String> = RefCell::new(String::new());
        
        // **** NEW CODE HERE ****
        static BALANCES : RefCell<HashMap<Principal, u128>> = RefCell::new(HashMap::new());

}

#[ic_cdk::init]
fn init(name: String, symbol: String) {
    NAME.with_borrow_mut(|cell| *cell = name);
    SYMBOL.with_borrow_mut(|cell| *cell = symbol);
}

#[ic_cdk::query]
fn name() -> String {
    NAME.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn symbol() -> String {
    SYMBOL.with_borrow(|cell| cell.clone())
}

// **** NEW CODE HERE ****
#[ic_cdk::query]
fn balance_of(principal: Principal) -> u128 {
    BALANCES.with_borrow(|cell| *cell.get(&principal).unwrap_or(&0))
}

ic_cdk::export_candid!();
```

### `TOTAL_SUPPLY` Tracks The Tokens in Circulation

The `TOTAL_SUPPLY` variable will keep track of the cumulative tokens minted. The minting and transfer function will be introduced in following section.

Add the `TOTAL_SUPPLY` variable into your thread local storage.

```rust
use std::{cell::RefCell, collections::HashMap};
use candid::Principal;

thread_local! {
        static NAME : RefCell<String> = RefCell::new(String::new());
        static SYMBOL : RefCell<String> = RefCell::new(String::new());
        static BALANCES : RefCell<HashMap<Principal, u128>> = RefCell::new(HashMap::new());
        
        // **** NEW CODE HERE ****
        static TOTAL_SUPPLY: RefCell<u128> = RefCell::new(0);
}

#[ic_cdk::init]
fn init(name: String, symbol: String) {
    NAME.with_borrow_mut(|cell| *cell = name);
    SYMBOL.with_borrow_mut(|cell| *cell = symbol);
}

#[ic_cdk::query]
fn name() -> String {
    NAME.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn symbol() -> String {
    SYMBOL.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn balance_of(principal: Principal) -> u128 {
    BALANCES.with_borrow(|cell| *cell.get(&principal).unwrap_or(&0))
}

ic_cdk::export_candid!();
```

To retrieve the token’s total supply, add the query function `total_supply()`:

```rust
use std::{cell::RefCell, collections::HashMap};
use candid::Principal;

thread_local! {
        static NAME : RefCell<String> = RefCell::new(String::new());
        static SYMBOL : RefCell<String> = RefCell::new(String::new());
        static BALANCES : RefCell<HashMap<Principal, u128>> = RefCell::new(HashMap::new());
        static TOTAL_SUPPLY: RefCell<u128> = RefCell::new(0);
}

#[ic_cdk::init]
fn init(name: String, symbol: String) {
    NAME.with_borrow_mut(|cell| *cell = name);
    SYMBOL.with_borrow_mut(|cell| *cell = symbol);
}

#[ic_cdk::query]
fn name() -> String {
    NAME.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn symbol() -> String {
    SYMBOL.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn balance_of(principal: Principal) -> u128 {
    BALANCES.with_borrow(|cell| *cell.get(&principal).unwrap_or(&0))
}

// **** NEW CODE HERE ****
#[ic_cdk::query]
fn total_supply() -> u128 {
    TOTAL_SUPPLY.with_borrow(|cell| *cell)
}

ic_cdk::export_candid!();
```

With the balance and total supply logic in place, we are now ready to mint tokens into existence.

## Only owner can `Mint()`

The minting logic will be exclusive to the owner, who will be the the contract’s deployer. This section is divided in two, adding the owner variable and implement the minting function.

### `OWNER` variable

Add an `OWNER` variable and have its value initialize through the constructor.

```rust
use std::{cell::RefCell, collections::HashMap};
use candid::Principal;
use ic_cdk::api::msg_caller;

thread_local! {
        static NAME : RefCell<String> = RefCell::new(String::new());
        static SYMBOL : RefCell<String> = RefCell::new(String::new());
        static BALANCES : RefCell<HashMap<Principal, u128>> = RefCell::new(HashMap::new());
        static TOTAL_SUPPLY: RefCell<u128> = RefCell::new(0);
        
        // **** NEW CODE HERE ****
        static OWNER : RefCell<Principal> = RefCell::new(Principal::anonymous());
}

#[ic_cdk::init]
fn init(name: String, symbol: String) {
    NAME.with_borrow_mut(|cell| *cell = name);
    SYMBOL.with_borrow_mut(|cell| *cell = symbol);
    
    
    // **** NEW CODE HERE ****
    OWNER.with_borrow_mut(|cell| *cell = msg_caller());
}

#[ic_cdk::query]
fn name() -> String {
    NAME.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn symbol() -> String {
    SYMBOL.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn balance_of(principal: Principal) -> u128 {
    BALANCES.with_borrow(|cell| *cell.get(&principal).unwrap_or(&0))
}

#[ic_cdk::query]
fn total_supply() -> u128 {
    TOTAL_SUPPLY.with_borrow(|cell| *cell)
}

ic_cdk::export_candid!();
```

Add an `owner()` query function that returns the owner’s `Principal`:

```rust
use std::{cell::RefCell, collections::HashMap};
use candid::Principal;
use ic_cdk::api::msg_caller;

thread_local! {
        static NAME : RefCell<String> = RefCell::new(String::new());
        static SYMBOL : RefCell<String> = RefCell::new(String::new());
        static BALANCES : RefCell<HashMap<Principal, u128>> = RefCell::new(HashMap::new());
        static TOTAL_SUPPLY: RefCell<u128> = RefCell::new(0);
        static OWNER : RefCell<Principal> = RefCell::new(Principal::anonymous());
}

#[ic_cdk::init]
fn init(name: String, symbol: String) {
    NAME.with_borrow_mut(|cell| *cell = name);
    SYMBOL.with_borrow_mut(|cell| *cell = symbol);
    OWNER.with_borrow_mut(|cell| *cell = msg_caller());
}

#[ic_cdk::query]
fn name() -> String {
    NAME.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn symbol() -> String {
    SYMBOL.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn balance_of(principal: Principal) -> u128 {
    BALANCES.with_borrow(|cell| *cell.get(&principal).unwrap_or(&0))
}

#[ic_cdk::query]
fn total_supply() -> u128 {
    TOTAL_SUPPLY.with_borrow(|cell| *cell)
}

// **** NEW CODE HERE ****
#[ic_cdk::query]
fn owner() -> Principal {
    OWNER.with_borrow(|cell| *cell)
}

ic_cdk::export_candid!();
```

### Only owner can `mint()` Function

The `mint()` function creates new tokens into circulation and assigns them to a specified account. To prevent unauthorized minting, we’ll add access control so that only the contract owner can call this function.

When `mint()` is executed, it

1. Increases the recipient’s balance and
2. updates the total supply to account for the newly created tokens.

`mint()` returns a `Result`. Prior to our usage of boolean to return values, we’ll use the `Result` type to add a variation of return values.

Insert the `mint()` function below, we’ll explain the code details afterwards.

```rust
use std::{cell::RefCell, collections::HashMap};
use candid::Principal;
use ic_cdk::api::msg_caller;

thread_local! {
        static NAME : RefCell<String> = RefCell::new(String::new());
        static SYMBOL : RefCell<String> = RefCell::new(String::new());
        static BALANCES : RefCell<HashMap<Principal, u128>> = RefCell::new(HashMap::new());
        static TOTAL_SUPPLY: RefCell<u128> = RefCell::new(0);
        static OWNER : RefCell<Principal> = RefCell::new(Principal::anonymous());
}

#[ic_cdk::init]
fn init(name: String, symbol: String) {
    NAME.with_borrow_mut(|cell| *cell = name);
    SYMBOL.with_borrow_mut(|cell| *cell = symbol);
    OWNER.with_borrow_mut(|cell| *cell = msg_caller());
}

#[ic_cdk::query]
fn name() -> String {
    NAME.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn symbol() -> String {
    SYMBOL.with_borrow(|cell| cell.clone())
}

#[ic_cdk::query]
fn balance_of(principal: Principal) -> u128 {
    BALANCES.with_borrow(|cell| *cell.get(&principal).unwrap_or(&0))
}

#[ic_cdk::query]
fn total_supply() -> u128 {
    TOTAL_SUPPLY.with_borrow(|cell| *cell)
}

#[ic_cdk::query]
fn owner() -> Principal {
    OWNER.with_borrow(|cell| *cell)
}

// **** NEW CODE HERE ****

#[ic_cdk::update]
fn mint(to: Principal, amount: u128) -> Result<String, String> {
    // access control - early return if not owner
    if !OWNER.with_borrow(|owner| *owner == msg_caller()) {
        return Err("Only owner can mint".to_string());
    }
    
    // credit new tokens to the mintee
    BALANCES.with_borrow_mut(|map| {
        *map.entry(to).or_insert(0) += amount;
    });
    
    // increment total supply
    TOTAL_SUPPLY.with_borrow_mut(|supply| *supply += amount);
    
    return Ok(format!("Minted {amount} tokens to {to}"));
}

ic_cdk::export_candid!();
```

The `mint()` function accepts two arguments: the receiver principal to mint to and the amount. The logic:

1. Checks if the `msg_caller()` is the owner, and returns an `Err` early if the caller is not the owner
2. It credits the recipient’s account by adding the minted amount to their existing balance. If the recipient has no existing entry in the balance map, a new entry is created with a starting balance of zero before applying the increment.
3. It increments the total supply.

## Transfer Tokens: `transfer(to, amount)`

Users of our token should be able to transfer their tokens. The `transfer(to, amount)` function below would deduct the specified amount from the caller’s balance and credit the corresponding amount to the recipient’s balance.

At this point, your `lib.rs` should compile with the following additions. This snippet assumes the same imports as earlier sections. Add the `transfer()` function below to `lib.rs`.

```rust
// Earlier Codes ... (Code Example Too long)

// **** NEW CODE HERE ****
#[ic_cdk::update]
fn transfer(to: Principal, amount: u128) -> bool {
    BALANCES.with_borrow_mut(|map| {
		    // get caller's principal
        let from = msg_caller();
        
        // get caller's balance
        let from_bal = *map.get(&from).unwrap_or(&0);
        
        // check if balance is enough
        if from_bal < amount {
		        // return false if they do not have enough
            return false;
        }
				
				// 
        *map.entry(from).or_insert(0) -= amount;
        *map.entry(to).or_insert(0) += amount;
        true
    })
}

ic_cdk::export_candid!();
```

**`transfer` function logic:**

1. Retrieves the caller’s current balance, defaulting to `0` if no entry exists.
2. Returns `false` if the caller’s balance is insufficient for the transfer.
3. Deducts the transfer amount from the caller’s balance and adds it to the recipient’s balance in the same operation.

Because this canister uses `init` arguments, redeploying without `--mode=reinstall` will preserve old state. Re-deploy `token_canister` with the command below so the `init()` function would work.

```rust
dfx deploy token_canister_backend —mode=reinstall
```

We’ve explained why the `-mode=reinstall` flag is needed in the _initialize canister state and canister upgrades_ article.

## Allowance and Token Delegation

`ICRC-1` provides normal transfers and `ICRC-2` standard extends `ICRC-2` by providing `approve`/`transfer_from` capabilities.

### `Allowance`: Nested HashMap Variable

Now, we’ll create an `ALLOWANCE` state variable to store how many tokens an owner has approved for a spender to use. We’ll implement it as a nested HashMap where the first key is the owner and the second key is the delegated spender, with the value representing the approved amount.

```rust
// Earlier Codes ... (Code Example Too long)

thread_local! {

		static NAME : RefCell<String> = RefCell::new(String::new());
    static SYMBOL : RefCell<String> = RefCell::new(String::new());
    static BALANCES : RefCell<HashMap<Principal, u128>> = RefCell::new(HashMap::new());
    static TOTAL_SUPPLY: RefCell<u128> = RefCell::new(0);
    static OWNER : RefCell<Principal> = RefCell::new(Principal::anonymous());

    // **** NEW CODE HERE ****		
    static ALLOWANCE: RefCell<HashMap<Principal, HashMap<Principal, u128>>> =
        RefCell::new(HashMap::new());
    // Solidity equivalent of allowance[owner][spender] = amount
}

// ...
```

### `approve()`: Delegate Transfer Authority To a Spender

Add an `approve()` function that lets a token owner grant a spender permission to transfer a specific amount on their behalf. It updates the `ALLOWANCE` mapping by recording how many tokens the spender is allowed to use.

```rust
// Earlier Codes ...

// **** NEW CODE HERE ****
#[ic_cdk::update]
fn approve(spender: Principal, amount: u128) {
		// Token owner = caller
		let owner = msg_caller();
		
		// Add allowance to spender
		ALLOWANCE.with_borrow_mut(|cell| {
				
				// First mapping: Establishes a relationship between Owner and Spender
				let owner_allowances = cell.entry(owner).or_default();
				
				// Second mapping: Sets the amount the Spender can transfer
				owner_allowances.insert(spender, amount);
		});
}
```

`ALLOWANCE` is a nested `HashMap`.

1. The caller’s principal is used as the first-level key. Calling `.entry(owner)` checks whether an inner map already exists for that caller. If it does, `.or_default()` returns the existing map; if not, it creates a new empty map of `spender → allowance`.
2. With this inner map in hand, we insert a key–value pair where the **spender** is the key and the **approved amount** is the value, recording how many tokens the spender is allowed to transfer on the caller’s behalf.

### Check The Spender’s Allowance: `allowance()`

The function below reads the nested `ALLOWANCE` Hashmap.

```rust
// Earlier Codes ...

// **** NEW CODE HERE ****

#[ic_cdk::query]
fn allowance(owner: Principal, spender: Principal) -> u128 {
    ALLOWANCE.with_borrow(|cell| {
        // Checks for a owner spender relationship
        let Some(inner) = cell.get(&owner) else {
            // If there is no relationship, then return zero
            return 0;
        };

        // If the there is an entry, check the allowance for spender
        *inner.get(&spender).unwrap_or(&0)
    })
}
```

**`allowance()` code snippet explanation:**

1. Checks if the `owner` has an entry within the `spender`. If it doesn’t, then it the `owner` never `approved` the spender, hence it immediately returns a **0**.
2. If the `owner` and `spender` entry exists, it moves on to get the second mapping of the spender to the allowance amount and returns the amount if there’s an entry.

### `transferFrom()`: Transfer Delegated Tokens

The `transfer_from` function below allows an approved spender to transfer tokens from the owner to a receiver. It:

1. Checks if the `msg_caller()` (spender) was allocated an allowance by the owner (`from`), if it doesn’t then it returns false early.
2. Next, we check if the specified `amount` exceeds the allowance, if it does it returns false early.
3. Then it checks that the owner has sufficient balance for the transfer amount.
4. If the conditions above checks out, it credits the recipient with the amount and debits the owner.
5. Lastly, it reduces the spending allowance of `msg_caller()`.

```rust
// Earlier Codes ...

// **** NEW CODE HERE ****
#[ic_cdk::update]
fn transfer_from(from: Principal, to: Principal, amount: u128) -> bool {
    let spender = msg_caller();

    // check allowance
    let allowed = ALLOWANCE.with_borrow(|cell| {

        cell.get(&from)
            .and_then(|inner| inner.get(&spender).copied())
            .unwrap_or(0)
    });
    
    if allowed < amount {
        return false;
    }

    // check balance
    let from_bal = BALANCES.with(|cell| {
        let map = cell.borrow();
        map.get(&from).copied().unwrap_or(0)
    });
    
    // return false early
    if from_bal < amount {
        return false;
    }

    // update balances
    BALANCES.with(|cell| {
        let mut map = cell.borrow_mut();

        // debit `from`
        map.insert(from, from_bal - amount);

        // credit `to`
        let to_bal = map.get(&to).copied().unwrap_or(0);
        map.insert(to, to_bal + amount);
    });

    // consume allowance (and optionally tidy zeros)
    ALLOWANCE.with(|cell| {
        let mut outer = cell.borrow_mut();
        if let Some(inner) = outer.get_mut(&from) {
            let entry = inner.entry(spender).or_insert(0);
            *entry -= amount; // safe due to allowed >= amount

            if *entry == 0 {
                inner.remove(&spender);
            }
            if inner.is_empty() {
                outer.remove(&from);
            }
        }
    });

    true
}
```

## Testing Suite : RareSkills Tokens

The method we’ll use to test **token\_canister** is through switching developer identities in dfx to simulate multiple user interactions.

Deploy **token\_canister**,

```rust
dfx deploy
```

then pass “RareSkills” and “RS” as the arguments in this order.

![Screenshot 2025-08-20 at 20.38.14.png](../.gitbook/assets/Screenshot_2025-08-20_at_20.38.14.png)

We haven’t minted any tokens yet, to make it interesting let’s mint tokens to a new Principal, `Alice`.

### Create a New Developer Identity in dfx

To create a new private-public key pair in dfx, use the command below:

```rust
dfx identity new <Identity_Name>
```

As an example, replace `<Identity_Name>` with `Alice`. Run.

```rust
dfx identity new Alice
```

Although we’ve created a new identity, `Alice` has not been configured by dfx to be used. We can confirm it by running.

```rust
dfx identity whoami
```

dfx should still be using the default generated identity.

To use `Alice`, run the command below

```rust
dfx identity use Alice
```

Then check that dfx is using Alice with the `dfx identity whoami` command.

`Alice`'s principal or address on each of your developer environment should be different from this example’s. As an example we’ll associate `Alice` to this principal: `p67cs-wteuw-2m25n-457nx-dxwq7-uk36c-qlodh-nh5dw-uhzx4-ga6k3-yae`.

We can create as many identities as needed. The default Identity should have the name `default`. View the list of dfx identities using:

```rust
dfx identity list
```

![Screenshot 2025-08-20 at 21.15.45.png](../.gitbook/assets/Screenshot_2025-08-20_at_21.15.45.png)

### Mint RareSkills Tokens

The owner of the canister should be under `default`’s principal since we were using that identity to deploy **token\_canister**.

Switch back to `default` identity and call the `mint()` function.

```bash
dfx canister call token_canister_backend mint
```

dfx would prompt you to pass the `to : Principal` and `amount : u128`.

We’ll mint 1000 RareSkills Tokens to `Alice`, therefore pass `p67cs-wteuw-2m25n-457nx-dxwq7-uk36c-qlodh-nh5dw-uhzx4-ga6k3-yae` (use your Alice Identities principal, this is an example) and `1000`.

![Screenshot 2025-08-20 at 21.32.43.png](../.gitbook/assets/Screenshot_2025-08-20_at_21.32.43.png)

Call the `balance_of()` function to verify `Alice`’s balance.

![Screenshot 2025-08-20 at 21.33.57.png](../.gitbook/assets/Screenshot_2025-08-20_at_21.33.57.png)

### Transfer RareSkills Tokens

Switch to Alice identity and transfer 250 tokens to this address : `hpikg-6exdt-jn33w-ndty3-fc7jc-tl2lr-buih3-cs3y7-tftkp-sfp62-gqe`.

```bash
dfx canister call token_canister_backend transfer
```

![Screenshot 2025-08-20 at 21.38.34.png](../.gitbook/assets/Screenshot_2025-08-20_at_21.38.34.png)

Cross-check `Alice’s` balance and the random principal’s balance:

Alice would have a remaining balance of 750.

![Screenshot 2025-08-20 at 21.39.39.png](../.gitbook/assets/Screenshot_2025-08-20_at_21.39.39.png)

And `hpikg-6exdt-jn33w-ndty3-fc7jc-tl2lr-buih3-cs3y7-tftkp-sfp62-gqe` a new balance of 250.

![Screenshot 2025-08-20 at 21.40.20.png](../.gitbook/assets/Screenshot_2025-08-20_at_21.40.20.png)

### Approve Spender

The `approve(spender, amount)` function gives authority to a spender to transfer tokens on behalf of the `msg_caller()`.

Since we’re currently using Alice, we’ll approve default to spend all of its balance and transfer it to the random Identity again (`hpikg-6exdt-jn33w-ndty3-fc7jc-tl2lr-buih3-cs3y7-tftkp-sfp62-gqe`).

Call approve and pass `default`'s principal as the first argument and `750` as the second.

```bash
dfx canister call token_canister_backend approve
```

Then switch dfx identities to `default` and invoke the `transfer_from()` function pass:

* `From` : Alice’s Principal
* `To` : `hpikg-6exdt-jn33w-ndty3-fc7jc-tl2lr-buih3-cs3y7-tftkp-sfp62-gqe`
* `Amount`: 750

![Screenshot 2025-08-20 at 22.11.03.png](../.gitbook/assets/Screenshot_2025-08-20_at_22.11.03.png)

Check that the random identity’s balance is now `1000`

![Screenshot 2025-08-20 at 22.10.33.png](../.gitbook/assets/Screenshot_2025-08-20_at_22.10.33.png)

And Alice’s is now `0`.

![Screenshot 2025-08-20 at 22.12.00.png](../.gitbook/assets/Screenshot_2025-08-20_at_22.12.00.png)

### ICP Javascript Agent

We have learnt how to interface with `token_canister` using dfx, the next article teaches how to use ICP Javascript Agent, similar to Ethers.js, to interact with token\_canister, and in extension other canisters.
