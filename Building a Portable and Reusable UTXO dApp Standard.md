Building A Portable and Reusable UTXO dApp Standard
===================================================

As some of you may recall, I came up with an idea for a [universal dApp interface/package standard](https://www.ergoforum.org/t/universal-dapp-interface-package-standard/153) about a year ago. This idea was quite cool, yet naive in many aspects due to having little experience developing dApps on top of Ergo at that point.

Now that we're progressing forward with some real dApps deployed, and more in the works, I've re-envisioned the original idea into something that I think is achievable in the near future and has pretty much all of the same benefits of the original idea.

Today we have Rust Ergo-Lib(aka. sigma-rust), a great library that is being developed by @greenden which provides us with the vast majority of tools we need for writing off-chain dApp code. Furthermore, Ergo-Lib already has WASM bindings, with mobile bindings in the works as well. What this means is that off-chain dApp code we write in Rust is reusable for effectively every target we desire.

Thus we have the tools to write off-chain code once as a Rust library of pure functions, and allow effectively any wallet in the entire ecosystem to use said library to implement a front-end for the dApp. As such we are building performant Portable and Reusable dApps with all of the complex off-chain logic handled in a single codebase which is abstracted away from the front-end implementer.

Furthermore, this off-chain Rust code can be modeled similarly to [EIP-6 The Informal Spec Standard](https://github.com/ergoplatform/eips/blob/master/eip-0006.md). What's extremely valuable with this approach is that based on the spec you write for your dApp, you can also encode predicate checkers which verify every single input UTXO which is provided to the off-chain dApp library Action methods is indeed a valid box at a given stage. As such we have guarantees that the Action(which creates a new UnsignedTx) will 100% work because the inputs we provide are guaranteed to be valid.

This was the original idea behind the [Ergo Protocol Framework](https://github.com/robkorn/ergo-utilities-rust/tree/EPF-v0.1/ergo-protocol-framework). This is a library which allows you to convert your informal spec easily into Rust structs/code, while building these predicate checkers to guarantee input validity. The current Ergo Protocol Framework (v0.1) shows how this is possible, however it is currently deprecated for the goal at hand due to the fact that WASM compilation does not support complex type-level features which were originally implemented in v0.1. In the coming future I will be working on creating a macro interface for v0.2 which allows developers to perform the same tasks/acquire the same benefits, but uses much simpler types which have no problem compiling to WASM and being used in JS.

That said, the EPF isn't required to get all of the benefits of a PaR Ergo dApp, but it is an approach to simplify and solidify such a path for developers. Projects coming out in the near future will provide an example of how this is possible without the EPF, but in a slightly more manual manner.

With the Ergo Protocol Framework-based approach to building off-chain dApp code, we have all of the logic for performing state transitions(Actions/building txs) based on provided inputs. However one block of the puzzle here that is missing is how the wallet integrator of the dApp actually acquires the inputs UTXOs?

Currently the best thing as an ecosystem that we have to fill this role as a standard is [EIP-1 UTXO-Set Scanning Wallet API](https://github.com/ergoplatform/eips/blob/master/eip-0001.md). This however is limited to being used with the full node, and currently is very low-level/manual. This makes it's usefulness quite restricted to certain usecases.

As such I have a proposal for the next step forward for building truly Portable and Reusable UTXO-based dApps. For the next version of the Ergo Protocol Framework, we need to improve the ability to precisely specify the valid contents of a box at a given Stage (and for more general predicated box). From this specification (which uses Rust Ergo-Lib), we will then be able to automatically extract the details for how to acquire said box from external sources.

As such, once you define your `Stage`, you have effectively created a spec in Rust of what that stage is. Thus from there we can automatically generate and expose an interface that allows you to convert your `Stage` struct into a UTXO-scanning registration json. Example pseudocode:

```rust
let scan_json = MyStage::register_scan_json();
```

Furthermore, once the Ergo Explorer Backend supports complex queries for searching for boxes in the UTXO-set, then we can also automatically generate the endpoint for the developer to retrieve the UTXO they need. Example pseudocode:

```rust
let endpoint = MyStage::explorer_endpoint("ip_here");
let mystage_box = request.get(endpoint);
```

Thus the front-end developer who is implementing the dApp can choose whether they want to use an Ergo Node + UTXO-set scanning, or they would like to use an Ergo Explorer Backend in order to acquire the UTXOs for the Actions. The goal here is to automate as much as possible, so that once the developer specifies a stage, the UTXO-set scan json and the explorer endpoint are automatically generated based off of the extracted data from the specified stage struct.


With all of that said, we have a realistic plan for building Portable and Reusable UTXO-based dApps in the near future. None of the pieces left to build are extremely challenging to solve, just generally a bit time consuming.

Building out such tooling would allow the standard process for building Ergo dApps be:

1. Create an informal spec following EIP-6.
2. Write your smart contract in ErgoScript which matches the spec.
3. Write your off-chain dApp library in Rust using the Ergo Protocol Framework which matches the spec.
4. Distribute the dApp library to all wallets/apps which wish to implement a frontend for your dApp.
5. Frontend devs use the automatically generated utxo-set scan json/explorer endpoints to acquire the UTXOs users need to perform Actions in the protocol.
6. Frontend devs build out the frontend on any platform they desire (desktop/browser extension/mobile).

