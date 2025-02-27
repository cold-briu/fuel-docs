# Writing A Sway Smart Contract

## Installation

<!-- This example should include the instructions for installing Rust & Fuelup-->
<!-- installation:example:start -->
Start by [installing the Rust toolchain](https://www.rust-lang.org/tools/install).

Then, [install the Fuel toolchain](https://github.com/FuelLabs/fuelup).
<!-- installation:example:end -->

<!-- This example should include the instructions for installing the latest toolchain-->
<!-- toolchain_installation:example:start -->
Make sure you have the latest version of `fuelup` by running the following command:

```console
$ fuelup self update
Fetching binary from https://github.com/FuelLabs/fuelup/releases/download/v0.18.0/fuelup-0.18.0-aarch64-apple-darwin.tar.gz
Downloading component fuelup without verifying checksum
Unpacking and moving fuelup to /var/folders/tp/0l8zdx9j4s9_n609ykwxl0qw0000gn/T/.tmpiNJQHt
Moving /var/folders/tp/0l8zdx9j4s9_n609ykwxl0qw0000gn/T/.tmpiNJQHt/fuelup to /Users/.fuelup/bin/fuelup
```

Then run `fuelup toolchain install beta-3` to install the `beta-3` toolchain.

Finally, set the `beta-3` toolchain as your default distribution with the following command:

```console
$ fuelup default beta-3
default toolchain set to 'beta-3-aarch64-apple-darwin'
```

You can check your current toolchain anytime by running `fuelup show`.
<!-- toolchain_installation:example:end -->

> Having problems with this part? Post your question on our forum [https://forum.fuel.network/](https://forum.fuel.network/). To help you as efficiently as possible, include the output of this command in your post: `fuelup show.`

## Your First Sway Project

We'll build a simple counter contract with two functions: one to increment the counter, and one to return the value of the counter.

**Start by creating a new, empty folder. We'll call it `fuel-project`.**

### Writing the Contract

Create a contract project inside of your `fuel-project` folder:

```sh
$ cd fuel-project
$ forc new counter-contract
To compile, use `forc build`, and to run tests use `forc test`

----

Read the Docs:
- Sway Book: https://fuellabs.github.io/sway/latest
- Rust SDK Book: https://fuellabs.github.io/fuels-rs/latest
- TypeScript SDK: https://fuellabs.github.io/fuels-ts/

Join the Community:
- Follow us @SwayLang: https://twitter.com/SwayLang
- Ask questions on Discourse: https://forum.fuel.network/

Report Bugs:
- Sway Issues: https://github.com/FuelLabs/sway/issues/new
```

<!-- This example should include a tree for a new forc project and explain the boilerplate files-->
<!-- forc_new:example:start -->
Here is the project that `Forc` has initialized:

```console
$ tree counter-contract
counter-contract
├── Forc.toml
└── src
    └── main.sw
```

`Forc.toml` is the _manifest file_ (similar to `Cargo.toml` for Cargo or `package.json` for Node) and defines project metadata such as the project name and dependencies.
<!-- forc_new:example:end -->

Open your project in a code editor and delete the boilerplate code in `src/main.sw` so that you start with an empty file.

Every Sway file must start with a declaration of what type of program the file contains; here, we've declared that this file is a contract.

```sway
{{#include ../../quickstart-example/counter-contract/src/main.sw:contract}}
```

Next, we'll define a storage value. In our case, we have a single counter that we'll call `counter` of type 64-bit unsigned integer and initialize it to 0.

```sway
{{#include ../../quickstart-example/counter-contract/src/main.sw:storage}}
```

### ABI

An ABI defines an interface, and there is no function body in the ABI. A contract must either define or import an ABI declaration and implement it. It is considered best practice to define your ABI in a separate library and import it into your contract because this allows callers of the contract to import and use the ABI in scripts to call your contract.

For simplicity, we will define the ABI directly in the contract file.

```sway
{{#include ../../quickstart-example/counter-contract/src/main.sw:abi}}
```

### Implement ABI

Below your ABI definition, you will write the implementation of the functions defined in your ABI.

```sway
{{#include ../../quickstart-example/counter-contract/src/main.sw:counter-contract}}
```

> **Note**: `storage.counter` is an implicit return and is equivalent to `return storage.counter;`.

Here's what your code should look like so far:

File: `./counter-contract/src/main.sw`

```sway
{{#include ../../quickstart-example/counter-contract/src/main.sw:all}}
```

### Build the Contract

From inside the `fuel-project/counter-contract` directory, run the following command to build your contract:

```console
$ forc build
  Compiled library "core".
  Compiled library "std".
  Compiled contract "counter-contract".
  Bytecode size is 232 bytes.
```

Let's have a look at the content of the `counter-contract` folder after building:

```console
$ tree .
.
├── Forc.lock
├── Forc.toml
├── out
│   └── debug
│       ├── counter-contract-abi.json
│       ├── counter-contract-storage_slots.json
│       └── counter-contract.bin
└── src
    └── main.sw
```

We now have an `out` directory that contains our build artifacts such as the JSON representation of our ABI and the contract bytecode.

## Testing your Contract

We will start by adding a Rust integration test harness using a Cargo generate template. If this is your first time going through this quickstart, you'll need to install the `cargo generate` command. In the future, you can skip this step as it will already be installed.

Navigate to your contract and then run the installation command:

```console
$ cd counter-contract
changed directory into `counter-countract`
$ cargo install cargo-generate
 Updating crates.io index...
 installed package `cargo-generate v0.17.3`
```

> **Note**: You can learn more about cargo generate by visiting [its repository](https://github.com/cargo-generate/cargo-generate).

Now, let's generate the default test harness with the following:

```console
$ cargo generate --init fuellabs/sway templates/sway-test-rs --name counter-contract
⚠️   Favorite `fuellabs/sway` not found in config, using it as a git repository: https://github.com/fuellabs/sway
🤷   Project Name : counter-contract
🔧   Destination: /home/user/path/to/counter-contract ...
🔧   Generating template ...
[1/3]   Done: Cargo.toml
[2/3]   Done: tests/harness.rs
[3/3]   Done: tests
🔧   Moving generated files into: `/home/user/path/to/counter-contract`...
✨   Done! New project created /home/user/path/to/counter-contract
```

Let's have a look at the result:

```console
$ tree .
.
├── Cargo.toml
├── Forc.lock
├── Forc.toml
├── out
│   └── debug
│       ├── counter-contract-abi.json
│       ├── counter-contract-storage_slots.json
│       └── counter-contract.bin
├── src
│   └── main.sw
└── tests
    └── harness.rs
```

We have two new files!

- The `Cargo.toml` is the manifest for our new test harness and specifies the required dependencies including `fuels` the Fuel Rust SDK.
- The `tests/harness.rs` contains some boilerplate test code to get us started, though doesn't call any contract methods just yet.

Now that we have our default test harness, let's add some useful tests to it.

At the bottom of `test/harness.rs`, define the body of `can_get_contract_id()`. Here is what your code should look like to verify that the value of the counter did get incremented:

File: `tests/harness.rs`

```sway
{{#include ../../quickstart-example/counter-contract/tests/harness.rs:contract-test}}
```

Run `cargo test` in the terminal. If all goes well, the output should look as follows:

```console
$ cargo test
  ...
  running 1 test
  test can_get_contract_id ... ok
  test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.11s
```

## Deploy the Contract

It's now time to deploy the contract to the testnet. We will show how to do this using `forc` from the command line, but you can also do it using the [Rust SDK](https://github.com/FuelLabs/fuels-rs#deploying-a-sway-contract) or the [TypeScript SDK](https://github.com/FuelLabs/fuels-ts/#deploying-contracts).

In order to deploy a contract, you need to have a wallet to sign the transaction and coins to pay for gas. First, we'll create a wallet.

### Install the Wallet CLI

Follow [these steps to set up a wallet and create an account](https://github.com/FuelLabs/forc-wallet#forc-wallet).

After typing in a password, be sure to save the mnemonic phrase that is output.

With this, you'll get a fuel address that looks something like this: `fuel1efz7lf36w9da9jekqzyuzqsfrqrlzwtt3j3clvemm6eru8fe9nvqj5kar8`. Save this address as you'll need it to sign transactions when we deploy the contract.

### Get Testnet Coins

With your account address in hand, head to the [testnet faucet](https://faucet-beta-3.fuel.network/) to get some coins sent to your wallet.

### Deploy To Testnet

Now that you have a wallet, you can deploy with `forc deploy` and passing in the testnet endpoint like this:

`forc deploy --node-url beta-3.fuel.network/graphql --gas-price 1 --random-salt`

> **Note**: We set the gas price to 1. Without this flag, the gas price is 0 by default and the transaction will fail.

The terminal will ask for the address of the wallet you want to sign this transaction with, paste in the address you saved earlier, it looks like this: `fuel1efz7lf36w9da9jekqzyuzqsfrqrlzwtt3j3clvemm6eru8fe9nvqj5kar8`

The terminal will output your `Contract id` like this:

```console
Contract id: 0xd09b469b0c31c05222b553021aa23c3b6a535db5092c22b84690dc88ca17deaa
```

Be sure to save this as you will need it to build a frontend with the Typescript SDK later in this tutorial.

The terminal will output a `Transaction id to sign` and prompt you for a signature. Open a new terminal tab and view your accounts by running `forc wallet accounts`. If you followed these steps, you'll notice you only have one account, `0`.

Grab the `Transaction id to sign` from your other terminal and sign with your account by running the following command:

```console
forc wallet sign --account `[the account index, without brackets]` tx-id `[Transaction id to sign, without brackets]`
```

Your command should look like this:

```console
$ forc wallet sign --account 0 tx-id 16d7a8f9d15cfba1bd000d3f99cd4077dfa1fce2a6de83887afc3f739d6c84df
Please enter your password to decrypt initialized wallet's phrases:
Signature: 736dec3e92711da9f52bed7ad4e51e3ec1c9390f4b05caf10743229295ffd5c1c08a4ca477afa85909173af3feeda7c607af5109ef6eb72b6b40b3484db2332c
```

Enter your password when prompted, and you'll get back a `signature`. Save that signature, and return to your other terminal window, and paste that in where its prompting you to `provide a signature for this transaction`.

Finally, you will get back a `TransactionId` to confirm your contract was deployed. With this ID, you can head to the [block explorer](https://fuellabs.github.io/block-explorer-v2/) and view your contract.

> **Note**
> You should prefix your `TransactionId` with `0x` to view it in the block explorer

![block explorer](../images/block-explorer.png)

### Congrats, you have completed your first smart contract on Fuel ⛽

[Here is the repo for this project](https://github.com/FuelLabs/quickstart). If you run into any problems, a good first step is to compare your code to this repo and resolve any differences.

Tweet us [@fuel_network](https://twitter.com/fuel_network) letting us know you just built a dapp on Fuel, you might get invited to a private group of builders, be invited to the next Fuel dinner, get alpha on the project, or something 👀.

## Need Help?

Get help from the team by posting your question in the [Fuel Forum](https://forum.fuel.network/).
