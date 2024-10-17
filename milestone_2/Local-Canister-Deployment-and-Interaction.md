# Local Canister Deployment and Interaction

![ICP New 2.jpg](Local%20Canister%20Deployment%20and%20Interaction/ICP_New_2.jpg)

This tutorial guides you through deploying a Canister locally. Just as tools like Foundry, Hardhat, and Remix assist in building and testing Ethereum smart contracts off-chain, there is a similar set tools available for deploying and testing Canisters locally. Comparably:

- Remix IDE = `Motoko Playground`
- Foundry / Hardhat = `DFX`

ICP has yet to have a public testnet like Sepolia since deployment and transaction fees are already low. Hence, to effectively deploy for 'free' is by using the alternatives mentioned above.

### Outline

- `Motoko Playground`: A web-based IDE like Remix
- The ICP equivalent of Foundry & Hardhat: `DFX`
- Interacting with Your Locally Deployed Canister

## `Motoko Playground`: A web-based IDE like Remix

`Motoko Playground` is an online IDE that compiles and test canisters written in the Motoko Language. It functions similarly to Remix. We’ll start with deploying a “Hello World” Canister on Motoko Playground.

Visit the [Motoko Playground website](https://m7sm4-2iaaa-aaaab-qabra-cai.raw.ic0.app). 

You should be greeted with several project templates to get started on, choose the `New Motoko project`.

![Screenshot 2024-10-08 at 13.04.31.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-08_at_13.04.31.png)

Just like Remix, we can write the smart contract on the `editor section` and deploy it.

![Screenshot 2024-10-08 at 13.13.54.png](Local%20Canister%20Deployment%20and%20Interaction/b5cbe39a-e628-4359-91bb-d466832b9bc9.png)

Within the `Editor`, copy paste the following codes written in **Motoko**:

```jsx
actor HelloWorld {
  public query func hi() : async Text {
    return "Helo World!";
  };
};
```

The contract above in Solidity translates to:

```jsx
contract HelloWorld {
    function hi() public pure returns (string memory) {
        return "Hello World!";
    }
}
```

The syntax may differ a bit, but from an overall point of view, they have a similar feel. We’ll explore the use of the language more in-depth in the next section.

Copy pasted the `HelloWorld` actor contract, hit the deploy button on the top right.

![Screenshot 2024-10-08 at 13.31.55.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-08_at_13.31.55.png)

Next, hit `Install`. This will compile your code and deploy it.

![Screenshot 2024-10-08 at 13.33.23.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-08_at_13.33.23.png)

Once successfully deployed, you will be served the `Candid UI`, this is how we can interface with the actor function. The `Query` call invokes the smart contract function and returns the hardcoded string “Hello World!”.

![Screenshot 2024-10-08 at 13.35.09.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-08_at_13.35.09.png)

`Candid UI`, is comparable to the `Deployed Contracts` section in Remix IDE. It’s an automatically generated interface that allows us to easily interact with the canister functions.

![Screenshot 2024-10-12 at 20.01.57.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_20.01.57.png)

[Motoko Playground deploys canisters **online**](https://medium.com/dfinity/introducing-the-motoko-playground-an-online-development-environment-for-the-internet-computer-efb4cd09ea8b) directly to the Internet Computer (IC) network, making it ideal for quick prototyping and learning. However, it has limitations such as a 20 minute time limit and the inability to simulate cycles transaction. For  a more complete developer experience, using **`dfx`** is as a more suitable.

## The ICP equivalent of Foundry & Hardhat: `DFX`

The `DFX` tool provides a developer environment that closely mirrors the IC mainnet. It allows you to start a local node replica, compile your canister smart contracts, and deploy them on the local replica.

### Install `DFX` on `Mac` or `Linux`

If you are running on on an Apple M1 or M2 CPU Chip, install `Rosetta` with the command below:

```jsx
softwareupdate --install-rosetta
```

Otherwise, you can immediately install `dfxvm`, a command line tool for running and managing `dfx`.

```jsx
sh -ci "$(curl -fsSL https://internetcomputer.org/install.sh)"
```

Next, choose the default option.

![Screenshot 2024-10-12 at 20.21.59.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_20.21.59.png)

Upon success, you should see the similar output

![Screenshot 2024-10-12 at 20.26.33.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_20.26.33.png)

Restart your terminal and enter `dfx`.

```bash
 dfx
```

You should see the following output that confirms `dfx` is installed on your machine.

![Screenshot 2024-10-12 at 20.27.47.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_20.27.47.png)

### Install DFX on `Windows`

You can install `dfx` on `docker` or on a cloud platform like `Github Codespaces`/ `Gitpod`.

- [**Install `dfx` on Docker.**](https://internetcomputer.org/docs/current/developer-docs/getting-started/install/#installing-dfx-via-dfxvm)
- To get started on `Github Codespaces` / `Gitpod`, choose one of the link:
    - `Github Codespaces`
        - [Rust Hello World](https://internetcomputer.org/docs/current/developer-docs/getting-started/hello-world#create-a-new-project)
        - [Motoko Hello World](https://internetcomputer.org/docs/current/developer-docs/getting-started/hello-world#create-a-new-project)
    - `Gitpod`
        - [Rust Hello World](https://gitpod.io/#https://github.com/dfinity/icp-hello-world-rust)
        - [Motoko Hello World](https://gitpod.io/#https://github.com/dfinity/icp-hello-world-motoko)

### Starting a local node

To start your local node, enter the command below:

```jsx
dfx start --clean --background
```

![Screenshot 2024-10-12 at 21.44.58.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_21.44.58.png)

This will run an local node instance that we can deploy our canister to using `dfx`.

### Hello World Canister using Motoko and Rust

We'll create a new project called `HelloWorld` using **DFX** and deploy it locally. *(If you are using Gitpod or Github Codespaces , simply run `dfx deploy` on you terminal and [skip this section](https://www.notion.so/Local-Canister-Deployment-and-Interaction?pvs=21))*

To get started, open a terminal and follow the steps below. This will generate a basic template to kickstart your canister development.

```jsx
dfx new HelloWorld
```

The setup process will guide you through a series of prompts, where you'll make a few selections to configure your canister settings—much like setting up a Next.js project. 

Next, select `Motoko` or `Rust` as your development language of choice

![Screenshot 2024-10-12 at 22.08.08.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_22.08.08.png)

Select `None` for the frontend framework:

![Screenshot 2024-10-12 at 22.08.55.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_22.08.55.png)

Next, select `Internet Identity` as the extra feature

![Screenshot 2024-10-12 at 22.18.10.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_22.18.10.png)

Your dfx project has just been created!

![Screenshot 2024-10-12 at 22.23.01.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_22.23.01.png)

Change directory to the newly created **HelloWorld** directory (`cd HelloWorld`) and open it up in VSCode or your code editor of choice.

![Screenshot 2024-10-12 at 22.24.43.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_22.24.43.png)

Run the following command in terminal where your project folder is: `path/HelloWorld`. This will deploy your canister to your local node! 

```bash
dfx deploy 
```

![Screenshot 2024-10-12 at 22.39.29.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_22.39.29.png)

To know what each of the files in the folder correspond to, make sure to read [Exploring the default project structure](https://internetcomputer.org/docs/current/tutorials/developer-journey/level-0/intro-dfx#exploring-the-default-project-structure).

## Interacting with Your Locally Deployed Canister

By default, the project we deployed has the following code template:

```jsx
actor {
  public query func greet(name : Text) : async Text {
    return "Hello, " # name # "!";
  };
};
```

We’ll explore three ways in which we can call the `greet` function and pass a string parameter using:

- The Backend `Candid UI`
- Candid UI For `Github Codespaces` / `Gitpod`
- The `DFX CLI`
- An `Agent`.js script

### Candid UI

To access the backend `Candid UI`, head over to the link that is printed in the terminal upon the successful deployment of the project titled Backend canister via `Candid Interface`.

![Screenshot 2024-10-12 at 22.39.29.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-12_at_22.39.29.png)

It should take you to the Candid UI, a locally hosted frontend that can be used to easily interact with your canister functions and see the output log and the call time response.

![Screenshot 2024-10-15 at 09.51.10.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-15_at_09.51.10.png)

### Candid UI For `Github Codespaces` / `Gitpod`

In `Github Codespaces` or `Gitpod`, to access the Candid UI, head over to the `port` section

![Screenshot 2024-10-16 at 19.36.33.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-16_at_19.36.33.png)

Typically, canisters will be hosted on the 4943 port. To access it, first make the visibility public by right clicking the `port` and **change its visibility**. Then, simply replace the **local host address** with the **Forwarded Address**. 

For example, your `Backend Candid UI` is hosted at: http://127.0.0.1:4943/?canisterId=be2us-64aaa-aaaaa-qaabq-cai&id=bkyz2-fmaaa-aaaaa-qaaaq-cai

and, the `Forwarded Address` is : https://redesigned-adventure-vxpgqrx6wv5f6g95-4943.app.github.dev/

Replace this section `http://127.0.0.1:4943/`, with the `Forwarded Address`. Hence making your final link:

https://redesigned-adventure-vxpgqrx6wv5f6g95-4943.app.github.dev/?canisterId=be2us-64aaa-aaaaa-qaabq-cai&id=bkyz2-fmaaa-aaaaa-qaaaq-cai

Don’t forget to make the visibility public or you will not be able to access it from your browser.

### DFX CLI

The `DFX` command-line interface (CLI) provides a direct way to interact with your canister. By default, the provided template looks like: 

```bash
dfx canister call <Canister-ID> <Function Name>
```

You can call the `greet` function using the command above by replacing the terms with the actual values of the canister id and function name:

```bash
dfx canister call bkyz2-fmaaa-aaaaa-qaaaq-cai greet
```

This will return the personalized greeting from the canister.

![Screenshot 2024-10-16 at 18.58.40.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-16_at_18.58.40.png)

If there are parameters, `dfx` will automatically ask for it in a step-by-step sequence as show in the image above.

### Agent.js

For more programmatic control, you can create a script using `Agent.js` to call your canister. Here's a script that calls the greet function of the HelloWorld Canister:

```jsx
import { Actor, HttpAgent } from '@dfinity/agent';
import { idlFactory } from './HelloWorld.did.js';

// If you are using Github Codespaces or Gitpod, the host link is can be found in the Port Section. How to use the link is described in the Candid UI section above 
const agent = await HttpAgent.create({ host: "http://127.0.0.1:4943/" });

// Disable certificate verification for local development (optional, but necessary for local `dfx` development)
await agent.fetchRootKey();

// Create the actor with the agent and the canister ID
const helloWorld = Actor.createActor(idlFactory, {
    agent,
    canisterId: "bkyz2-fmaaa-aaaaa-qaaaq-cai",
});

// Define and execute the greet function
async function greet(name) {
    const response = await helloWorld.greet(name);
    console.log(response);
}

greet("Your Name");

```

This script initializes an agent, connects to your canister using the `Actor`, and calls the `greet` function.

Run the script and you should get the following output:

![Screenshot 2024-10-16 at 19.52.25.png](Local%20Canister%20Deployment%20and%20Interaction/Screenshot_2024-10-16_at_19.52.25.png)