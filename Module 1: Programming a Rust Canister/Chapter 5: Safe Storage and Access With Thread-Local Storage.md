# Safe Storage and Access With Thread-Local Storage and RefCells

This article discusses how to safely store and access mutable variables for canisters written in Rust. 

In the previous chapters, we explained and demonstrated that immutable and mutable variables are declared using `static` and `static mut` (unsafe), respectively:

```rust
// Mutable
static mut Y: u128 = 41;

// Immutable
static X: bool = false;
```

Declaring immutable variables this way isn’t an issue; however, declaring mutable ones is.

When we declare a global variable using `static mut`, it creates a single memory location that is shared across the entire program and can be accessed from anywhere. Because this memory is globally accessible, Rust cannot determine when or where the variable is being read or modified, or whether multiple parts of the program are accessing it at the same time. Without this information, the compiler cannot enforce its usual ownership and borrowing rules that ensure memory safety.

As a result, the compiler cannot guarantee that accessing a `static mut` variable is safe. In simpler terms, it cannot guarantee that the variable won’t be read from and written to simultaneously by different parts of the program. To make this risk explicit, Rust requires every read and write to a `static mut` variable to be wrapped in an `unsafe` block, signaling that the programmer is taking responsibility for ensuring the access is correct.

```rust
// Mutable, but unsafe
static mut Y: u128 = 41;

#[ic_cdk::query]
fn read_y() -> u128 {
		// unsafe read
    unsafe { Y } 
}

#[ic_cdk::update]
fn set_y(new_y: u128) {
		// unsafe write
    unsafe { Y = new_y; } 
}
```

Although this pattern works, using `static mut` together with `unsafe` is unidiomatic and error-prone. It shifts the responsibility for correctness and memory safety entirely to the programmer and should be avoided unless you fully understand the implications.

Instead, the idiomatic and safe way to declare mutable static variables in Rust canisters is to use **thread-local storage** combined with **RefCell**. This approach avoids unsafe altogether and is the recommended best practice for managing canister state in Rust.

## Store `static` Variables Within The Thread-Local Storage

In Rust, a **thread-local** storage is a mechanism that gives each thread its own private copy of a variable. Instead of sharing one global memory location across the entire program, each thread works with its own isolated version. 

### Thread-Local Storage macro: `thread_local!`

A thread-local storage is declared with the `thread_local!` macro, with `static` variables placed inside it.

```rust
// thread local storage
thread_local! {

		// place our static variables within the macro
    static COUNTER: u8 = 0;
}
```

**Note**: Only `static` variables can be defined within `thread_local!`. 

Since thread-local storage creates an isolated copy of a static variable for each thread, different threads never read from or write to the same memory location. This isolation is what makes storing static variables in thread-local storage safe.

### Thread-local `static` variables

The variable `COUNTER` below is declared within the thread-local storage (`thread_local!`). Each thread that is spawned will get its own private copy of `COUNTER`, stored in a separate memory location. This copy is exclusive to that thread, so its data can’t be accessed or modified by any other thread. 

```rust
thread_local! {
    static COUNTER: u8 = 0;
}
```

If we have three threads, then each thread would have its own `COUNTER` variable that only they can increment. This way, different threads do not share the same variable, avoiding race conditions.

![Untitled-2025-06-09-1626.excalidraw-3.svg](Safe%20Storage%20and%20Access%20With%20Thread-Local%20Storage%20/Untitled-2025-06-09-1626.excalidraw-3.svg)

The examples above only showed immutable static variables. In the next section, we’ll see how to define *mutable* state using `thread_local!`, even though the `mut` keyword is not allowed inside the macro.

### Mutability for thread-local `static`s

Earlier, we learned that mutable global state in Rust is usually declared using `static mut`. However, this approach does not work for `static` variables declared inside the `thread_local!` macro. Thread-local `static` variables **cannot** use the `mut` keyword, and attempting to do so results in a compilation error:

```rust
thread_local! {
    static mut Z: u64 = 30; // ❌ Won’t compile
}
```

The `thread_local!` macro under the hood generates extra code for the `static` variables declared inside of it to safely manage per-thread storage. Therefore, we cannot add `mut` to the static variables. 

## Thread-Local `static`s use `RefCell` for mutability

To achieve mutability for static variables declared inside `thread_local!`, Rust requires a different approach: wrapping the variable’s type in a `RefCell` 

`RefCell` is an interior mutability type. It allows a value to be mutated through controlled, runtime-checked methods, which we’ll explore shortly. 

To declare a  `static`  mutable variable inside the thread-local storage, follow the instructions below:

1. First, import `RefCell` from the standard library:
    
    ```rust
    use std::cell::RefCell;
    ```
    
2. Wrap the `static` variable’s data type (denoted by the symbol `T`) with `RefCell`, `RefCell<T>`:
    
    ```rust
    thread_local! {
        static Z: RefCell<bool> 
    }
    ```
    
3. The `static` variable’s value is initialized with `RefCell::new(value)`:
    
    ```rust
    use std::cell::RefCell;
    
    thread_local! {
        static Z: RefCell<bool> = RefCell::new(true);
    }
    ```
    
    `RefCell::new(true)` initializes the `RefCell` with a boolean value of `true`. The value provided must match the type declared inside `RefCell<bool>`.
    

Variables using the `RefCell` wrapper are **mutable** at runtime, and access to the variable is controlled mainly through two methods:

- `.borrow()` for **read-only** operations and
- `.borrow_mut()` for **write** operations.

### Modifying a thread-local `static` using `RefCell` variable

Let’s demonstrate mutating the value of thread-local `static Z: RefCell<bool>`, which is a mutable state variable of type `bool`.

We have two functions:

- `set_z(_bool)` that sets an arbitrary **bool** value to `Z` and
- `get_z()` which reads the value of `Z`.

Don’t worry if the syntax looks strange — we’ll unpack it step by step. 

```rust
use std::cell::RefCell;

thread_local! {
    static Z: RefCell<bool> = RefCell::new(true);
}

// read z
#[ic_cdk::query]
fn get_z() -> bool {
    Z.with(|cell| *cell.borrow())
}

// mutate z
#[ic_cdk::update]
fn set_z(value: bool) {
    Z.with(|cell| *cell.borrow_mut() = value);
}

ic_cdk::export_candid!();

```

The calls may look confusing at first:

- `get_z()`: `Z.with(|cell| *cell.borrow()`.
- `set_z()`: `Z.with(|cell| *cell.borrow_mut() = value);`.

But there only three key parts:

- `Z.with()` gives us access to thread-local `Z`.
- The `|cell| cell` pattern is called a **closure**, which we’ll discuss in depth later. In short, it acts similarly to a callback function, where `|cell|` is a parameter that takes the variable `Z` as its argument. We then use `cell` to perform operations on `Z`.
- Since `Z` is wrapped in a `RefCell`, we are required to either use `.borrow()` for read-only access or `.borrow_mut()` for mutable access to reach the inner `bool` value.

There are two layers of access: `.with()` from `thread_local!` and `.borrow()`/`.borrow_mut()` from `RefCell`.

To keep things simple, we’ll set aside `thread_local!` and focus on learning how to access values within a `RefCell` variable. Later, we’ll return to a full example that places a `RefCell` inside `thread_local!`.

## Accessing values within `RefCell<T>`

A `RefCell<T>` is an immutable container that enables **interior mutability**: it allows us to read or modify the value it holds even when the `RefCell` itself is not declared as `mut`.

However, because the value is wrapped inside a `RefCell`, we cannot access it directly. Instead, Rust requires us to go through specific methods that enforce borrowing rules at runtime.

To understand how this works in practice, let’s start with a simple, local `RefCell` example.

### **Attempting to return a `RefCell` directly**

Consider the following function:

```rust
use std::cell::RefCell;

#[ic_cdk::query]
fn read_refcell() -> u8 {
		let x: RefCell<u8> = RefCell::new(20);
		// attempt to return the inner value
}
```

At first glance, you might expect to return `x` directly. However, doing so results in a compilation error:

```rust
use std::cell::RefCell;

#[ic_cdk::query]
fn read_refcell() -> u8 {
		let x: RefCell<u8> = RefCell::new(20);
		return x;
}
```

![Screenshot 2025-09-03 at 06.17.49.png](Safe%20Storage%20and%20Access%20With%20Thread-Local%20Storage%20/Screenshot_2025-09-03_at_06.17.49.png)

This fails because the function’s return type is `u8`, but `x` is a `RefCell<u8>`. Rust does not automatically extract the inner value for us.

To access the value stored inside a `RefCell`, we must explicitly borrow it.

### **Reading the inner value with `.borrow()`**

To read the value inside a `RefCell<T>`, we call `.borrow()`. This method returns a `Ref<T>`, which behaves like an immutable reference to the inner value.

If we try to return `x.borrow()` directly, we still get a type mismatch:

```rust
use std::cell::RefCell;

#[ic_cdk::query]
fn read_refcell() -> u8 {
		let x: RefCell<u8> = RefCell::new(20);
		return x.borrow();
}
```

![Screenshot 2025-08-14 at 22.52.26.png](Safe%20Storage%20and%20Access%20With%20Thread-Local%20Storage%20/Screenshot_2025-08-14_at_22.52.26.png)

This happens because `x.borrow()` returns a `Ref<u8>`, not a `u8`.

To obtain the actual `u8` value, we must **dereference** the `Ref<u8>` using `*`:

```rust
use std::cell::RefCell;

#[ic_cdk::query]
fn read_refcell() -> u8 {
		let x: RefCell<u8> = RefCell::new(20);
		*x.borrow()
}
```

At this point, the function correctly returns the inner value 20.

![Screenshot 2025-08-14 at 22.57.55.png](Safe%20Storage%20and%20Access%20With%20Thread-Local%20Storage%20/bb88c653-0956-4c34-9529-a46698e81541.png)

### **Writing to a RefCell with `.borrow_mut()`**

Reading from a `RefCell` uses `.borrow()`. To **modify** the inner value, we use `.borrow_mut()`, which provides mutable access.

In the example shown below, `x`'s value is changed from 20 to 40 using before `*x.borrow_mut() = 40;` returning its value.

```rust

use std::cell::RefCell;

#[ic_cdk::query]
fn write_refcell() -> u8 {
		let x: RefCell<u8> = RefCell::new(20);
		*x.borrow_mut() = 40;
		return *x.borrow();
}
```

The next section will show how to use `.with()` correctly.

### Accessing Thread-Local Variables with `.with()`

The usage of `.with()` is shown in the `get_x()` function below.

```rust
use std::cell::RefCell;

thread_local! {
    static X: RefCell<u8> = RefCell::new(40);
}

#[ic_cdk::query]
fn get_x() -> u8 {
    X.with(callback) // access thread-local variables using .with
}

//Call back function
fn callback(cell: &RefCell<u8>) -> u8 {
    *cell.borrow()
}

ic_cdk::export_candid!();
```

The `.with()` method takes a **callback function** as its argument. In the example, the callback function is `callback()`, so we call `.with(callback)`. 

`.with()` will hand over the responsibility of handling the `RefCell` variable to the `callback()` function. The `callback()` function must have a specific input and output syntax, which is:

1. It accepts a reference to the `RefCell` variable as its input: `cell: &RefCell<u8>`.
2. It returns the inner value of the `RefCell` variable, `u8`. 

Manipulating the thread-local `RefCell` variable happens through the input parameter `cell`, which is a reference to the thread-local `RefCell`. We  use `cell.borrow()` or `cell.borrow_mut()` to gain read or write access.

```rust
// Callback function
fn callback(cell: &RefCell<u8>) -> u8 { // cell is a reference to X
    
    // .borrow() returns Ref<u8>, so we dereference it to get the inner value
    *cell.borrow() 
}
```

### Using a Closure in Place of a Callback Function

You can replace the standalone `callback()` function with an inline **closure**. Closures in Rust are anonymous functions written directly at the call site, which keeps the code shorter and easier to follow.

```rust
use std::cell::RefCell;

thread_local! {
    static X: RefCell<u8> = RefCell::new(40); // ✅ mutable
}

#[ic_cdk::query]
fn get_x() -> u8 {
    X.with(|cell| *cell.borrow())  // closure is implemented instead of callback
}
```

**Closure syntax explained**

- `|cell|` defines the closure input parameter; here, it is a reference to the thread-local `RefCell<u8>.`
- Inside the closure, `.borrow()` creates an immutable borrow, and `*` dereferences it to yield the raw `u8`.
- We do not have to specify the input or the return type since they’re inferred by the compiler.

**Closures** behave the same as separate callback functions but are more practical because they are inline, avoid boilerplate, and let the compiler infer parameter and return types for you.

### Writing to a Thread-Local `RefCell`

Now, we demonstrate accessing a thread-local `static` variable of type `RefCell` using `.with()`, a closure (`|cell| cell...`)**,** and `.borrow()` & `.borrow_mut()` for read and write access.

The `set_x()` method below sets the value of `X` to any `u8` you provide:

```rust

use std::cell::RefCell;

thread_local! {
    static X : RefCell<u8> = RefCell::new(20);
}

#[ic_cdk::update]
fn set_x(a: u8) {
    X.with(|cell| *cell.borrow_mut() = a);
}

#[ic_cdk::query]
fn get_x() -> u8 {
    X.with(|cell| *cell.borrow())
}

ic_cdk::export_candid!();

```

Calling `set_x(255)`, followed by `get_x()` will return `255`.

![Screenshot 2025-08-15 at 13.40.43.png](Safe%20Storage%20and%20Access%20With%20Thread-Local%20Storage%20/Screenshot_2025-08-15_at_13.40.43.png)

The value of `X` is now **255**, from **20**, indicating that the state change was persisted.

To summarize up to this point, use `.borrow()` or `.borrow_mut()` to access the inner value of `RefCell` variables, and for `thread_local!` variables, you must also wrap that access inside a `.with()` call. 

### `.with()` combined with `.borrow()` and `.borrow_mut()`

There is a shortcut to calling a static thread-local `Refcell` variable. Instead of calling `.with()` and `.borrow()` or `.borrow_mut()` individually, we can combine them together:

- `.with()` + `.borrow()` = `.with_borrow()`,
- `.with()` + `.borrow_mut()` = `. with_borrow_mut()`,

The RefCell reference inside of the closure would, under the hood have the `.borrow()` method automatically called on them. The `COUNTER` canister below demonstrates using them:

```rust
use std::cell::RefCell;

thread_local! {
    static COUNTER : RefCell<u8> = RefCell::new(20);
}

#[ic_cdk::update]
fn increment() {

		// .with() + .borrow() = .with_borrow()
		COUNTER.with_borrow_mut(|cell| *cell += 1);
}

#[ic_cdk::query]
fn get_counter() -> u8 {

		// .with() + .borrow_mut() = .with_borrow_mut()
    COUNTER.with_borrow(|cell| *cell)
}

ic_cdk::export_candid!();
```

From this point forward, our code examples will use `.with_borrow()` and `.with_borrow_mut()` for simplicity.

In the next section we’ll learn about composite data types to declare state variables such as `Vec<T>`, and `HashMap<K, V>`.