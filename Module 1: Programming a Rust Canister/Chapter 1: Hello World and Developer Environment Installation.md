# Hello World and Developer Environment Installation

The goal of this tutorial is to set up your development environment and learn how to deploy your first ICP smart contract on your local network.

## Canister Smart Contracts

ICP refers to their smart contracts as *canisters*. Canisters store both logic and data, on-chain.

![canister.svg](Hello%20World%20and%20Developer%20Environment%20Installation/canister.svg)

The Internet Computer Protocol is WASM based — It utilizes a customized WebAssembly Virtual Machine (WASM VM) to run their canister smart contracts. 

Canisters can be written in any language that compiles to WASM or has an interpreter that targets WASM bytecode. However, official SDKs are currently available only for Rust, C++, Python, TypeScript, and Motoko (ICP’s native language). For this course, we will use **Rust** exclusively.

To begin, we’ll first install Rust and the developer environment, `dfx`, next.

## Rust Installation

Follow the official [Rust installation guide](https://www.rust-lang.org/tools/install). After you’ve installed Rust, verify the installation by running:

```bash
rustc --version
cargo --version
```

![Screenshot 2025-12-02 at 01.04.56.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2025-12-02_at_01.04.56.png)

### Install the WASM Build Target

After you have installed Rust in your local machine, install the **WebAssembly compilation target**  by running the following command:

```bash
rustup target add wasm32-unknown-unknown
```

Canisters written in Rust needs to be compiled into WebAssembly bytecode. The command above installs the `wasm32-unknown-unknown` target so Rust can compile your Rust canister code into a `.wasm` binary/bytecode.

## Setup The Developer Environment (`dfx`)

`dfx` is a **command-line tool (CLI)** that helps you build, deploy, and test canisters. 

### Windows

`dfx` is not natively supported on Windows. You will have to [install Linux on Windows](https://learn.microsoft.com/en-us/windows/wsl/install) ****and install `dfx` inside the Linux environment.

### Mac or Linux

Install `dfxvm`, the version manager for `dfx`, with following command:

```jsx
sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"
```

**Note**: If you’re on **Apple Silicon**, install `Rosetta` beforehand with the command `softwareupdate --install-rosetta`. Rosetta allows Apple Silicon Macs to run software built for intel-based Macs.

Choose the **default** configuration for `dfx`.

![Screenshot 2024-10-12 at 20.21.59.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2024-10-12_at_20.21.59.png)

Upon success, you should be presented with a output similar to the image below.

![Screenshot 2024-10-12 at 20.26.33.png](Hello%20World%20and%20Developer%20Environment%20Installation/7773f688-58d3-4322-bba1-f7739d45050d.png)

Restart your terminal and enter `dfx`:

```bash
 dfx
```

The following output confirms that `dfx` is installed on your machine.

![Screenshot 2024-10-12 at 20.27.47.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2024-10-12_at_20.27.47.png)

With `dfx` installed, we’ll begin with creating a `hello_world` canister and deploy it on the local testnet.

## Hello World

Run the command below to create a new `hello_world` Rust canister. 

```jsx
dfx new hello_world --type rust --no-frontend
```

- This command creates a directory called `hello_world`, which contains the default starter files you need to begin developing your canister.
- The `--type` flag specifies the programming language for your canister. In our case we are using **Rust**.
- The `—no-frontend` flag skips bootstrapping a default frontend website.

Navigate to the `hello_world` project directory and open it in your code editor of choice.

```jsx
cd hello_world
```

When you create a new Rust canister project with `dfx`, the main file where we write our canister code is at `lib.rs`. Navigate to the `lib.rs` file.

```bash
hello_world/
├──── src/
│    └──── hello_world_backend/
│         └──── src/
│              └── **lib.rs**
```

`lib.rs` will contain the following boilerplate code:

```rust
#[ic_cdk::query]
fn greet(name: String) -> String {
    format!("Hello, {}!", name)
}
```

The Rust code above is a public function `greet()`, which takes a string as its argument (`name`) and returns *“Hello”* concatenated with `name`. 

A detailed explanation of the code above and its underlying concepts is provided in the next article. For now, we’ll learn how to deploy a canister to the local testnet and call the its function.

## Deploy a Canister To The Local IC instance

To deploy the `hello_world` canister locally, we’ll need to start a local IC instance (localnet).

### Run an IC instance on your local machine

In a new terminal, input the command below. 

```jsx
dfx start
```

This will start your local IC instance and produce output similar to the image below:

![Screenshot 2025-04-10 at 15.29.08.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2025-04-10_at_15.29.08.png)

By default, the local IC instance runs on port **4943:** [**http://127.0.0.1:4943**](http://127.0.0.1:4943/_/dashboard). 

Once we have a local IC instance running, we can deploy our `hello_world` canister.

**Note**: To stop your local IC instance, use the command `Ctrl-C`, `dfx killall` or `dfx stop`.

### Deploy The `hello_world` Canister

To deploy the `hello_world` canister on the local IC instance, run the command below from the `hello_world` project directory:

```bash
dfx deploy 
```

A similar output to the one below indicates a successful deployment.

![Screenshot 2025-09-25 at 00.19.45.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2025-09-25_at_00.19.45.png)

The successful output of `dfx deploy` contains the **address** of your canister, and **a link to a canister interface frontend** where we can interact with its functions:

![Screenshot 2025-11-26 at 14.47.55.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2025-11-26_at_14.47.55.png)

### The Canister ID (smart contract address)

`hello_world`'s smart contract address or **Canister ID**, can be found at the successful terminal output, highlighted by the red box:

![Screenshot 2025-09-25 at 00.16.49.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2025-09-25_at_00.16.49.png)

The next section looks at how we can interact with the deployed `hello_world` canister.

## Methods To Interact With Your Canister

We’ll explore two ways in which we can call `hello_world`'s `greet()` function, and pass the `name` string parameter, which are through the:

- **Candid UI**, and
- **`dfx` CLI**

### The Candid UI

The **Candid UI** is an automatically generated and deployed frontend web interface to interact with your canister functions. It looks like this:

![Screenshot 2025-08-09 at 21.51.26.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2025-08-09_at_21.51.26.png)

The link to `Candid UI` can be found at the successful output of the `dfx deploy` command, titled **"Backend canister via Candid Interface**.**"** Open the link and it should take you to the **Candid UI**. 

![Screenshot 2025-09-25 at 00.22.47.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2025-09-25_at_00.22.47.png)

Try calling the `greet()` function and pass “RareSkills” as the string argument.

![Screenshot 2025-08-09 at 21.51.26.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2025-08-09_at_21.51.26.png)

**Note**: In local deployments with `dfx`, Canister IDs are [generated deterministically](https://github.com/dfinity/ic/blob/699e0c3351d44e7acdd2e743fede7c835b3b3558/rs/replicated_state/src/metadata_state.rs#L447), so the first, second, third, and subsequent canisters consistently receive the same Canister IDs across runs.

### The `dfx` CLI

`dfx` provides a direct way to interact with your canister through the command line interface (CLI). The following is a command format for calling a canister’s function:

```bash
dfx canister call <Canister-ID> <Function Name>
```

- `<Canister-ID>` can be either the actual ***Canister ID*** or the ***canister’s name**.*
- `<Function Name>` is the function signature.

We’ll demonstrate calling `greet()` using both the **Canister ID** and the **canister’s name**.

- **dfx canister call with the Canister ID**
    
    Replace the terms:
    
    - `<Canister-ID>` with the **Canister ID** found in the `dfx deploy` success terminal output. In the example’s case it’s `uxrrr-q777-77774-qaaaq-cai`.
    - `<Function Name>` with  `greet`
    
    ```bash
    dfx canister call uxrrr-q7777-77774-qaaaq-cai greet
    ```
    
    You will automatically be prompted for the function’s parameter in a step-by-step manner as shown in the image below.
    
    ![Screenshot 2025-06-30 at 20.15.53.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2025-06-30_at_20.15.53.png)
    

- **dfx canister call with the canister’s name**
    
    Optionally, you could use the canister’s name in place of `<Canister-ID>`**.** The format for a canister’s name is the project directory’s name (`hello_world`) + `_backend`: `hello_world_backend`:
    
    ```rust
    dfx canister call hello_world_backend greet
    ```
    
    The canister’s name can be found in the `dfx.json` file, highlighted by the red box. 
    
    ![Screenshot 2025-09-27 at 23.24.42.png](Hello%20World%20and%20Developer%20Environment%20Installation/Screenshot_2025-09-27_at_23.24.42.png)
    
    The `dfx.json` file is a configuration file that `dfx` uses to manage your canister project.
    

Now that we can deploy and test our canisters through `dfx`, the next article introduces how to write canister functions, including the code explanations for the default `greet()` function.