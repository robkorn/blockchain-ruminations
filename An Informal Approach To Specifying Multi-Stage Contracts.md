An Informal Approach To Specifying Multi-Stage Contracts
====
[Multi-stage contracts](https://link.springer.com/chapter/10.1007/978-3-030-31500-9_16) are a fascinating tool for implementing complex protocols on a UTXO-based system. They allow for great expressiveness and are generally approachable due to the fact that different phases of a protocol are cleanly separated into their own sub-contracts.

With that said, multi-stage contracts are extremely nascent as a model. This means that though they may be simpler when compared to alternatives, complexity is still going to be present when dealing with protocols that require numerous stages. Furthermore, we could benefit from conveying to others how a protocol is supposed to be implemented and/or work without writing a single line of code. As such it would be quite handy to have a framework to informally define specifications for our multi-stage contracts that is understandable, easily traversable, and can be used as a guide for eventually writing both the on-chain and the off-chain code.

With those being the outlined goals at hand, I will present a schema down below which uses plain english together with markdown to produce an informal specification for multi-stage contracts. But first, let us look at an example informal specification which uses said schema to define an NFT Auction Multi-Stage Contract.

---

# NFT Auction Multi-Stage Contract
This contract allows for a user to initiate an on-chain auction for a NFT/singleton token which accepts bids in Ergs. This informal specification is based off of the discussion/original design from [this Ergo Forum thread](https://www.ergoforum.org/t/auctions-on-ergo/122).


## ToC
1. [Bootstrap Stage](<#bootstrap-stage>)
2. [Auction Stage](<#auction-stage>)


## Bootstrap Stage

This stage allows a user to initiate the auction. It requires that they provide their NFT and a box with Erg which represents the starting bid. There are no hard-coded values, registers, or path conditions because this is the bootstrap stage and thus there is no contract yet.

### List Of Paths
- [Initiate Auction Path](<#Initiate-Auction-Path>)


### Initiate Auction Path

#### Inputs
1. Box which holds the NFT that is to be put up for auction.
2. Box with Ergs equivalent to the starting bid price.


#### Outputs
1. A box locked under the Auction contract/stage that holds the NFT and Ergs from the inputs, which also stores the auction holder's public key in register R4.


#### Leads To
[Auction Stage](<#Auction-Stage>)

---


## Auction Stage
In this stage the NFT is held up for auction and anyone who has enough Ergs can place a bid before the auction ends. All new bids must increment by at least 0.5 Erg.

### Hard-coded Values
- Auction holder's public key
- Auction end block height

### Registers
- R4: The current highest bidder's public key

### List Of Paths
- [Bid Path](<#Bid-Path>)
- [Conclude Auction Path](<#Conclude-Auction-Path>)


### Bid Path
This path recurses the box back into the Auction stage with the new Ergs held inside and the current winning bidder in R4 updated.

#### Inputs
1. The auction box.
2. Box(es) owned by the bidder which has enough Ergs to total the previous bid + 0.5 Erg.


#### Outputs
1. A new box locked under the Auction contract again which has the new bidder's public key in R4, the NFT, and the Ergs for the bid.
2. A box owned by the previous bidder which gives them back the Ergs which they bid.


#### Path Conditions
1. The current block height must be less than the auction end block height hard-coded into the contract.
2. The first output box must be locked under the Auction contract and holds the bidder's public key in R4, the NFT, and Ergs totalling at least 0.5 Erg more than the previous bid.
3. The second output box is owned by the previous bidder and has a total number of Ergs equal to their bid.


#### Leads To
[Auction Stage](<#Auction-Stage>)


### Conclude Auction Path

This path allows for the NFT and the funds to be redeemed to the auction winner and holder respectively once the auction finish height has been reached. If no one bid on the auction then the auction holder still has their public key in register R4, and therefore can redeem both their NFT and their Ergs to themselves.

#### Inputs
1. The auction box.


#### Outputs
1. A box owned by the address stored in R4 which holds the NFT.
2. A box owned by the auction holder which receives all of the Ergs bid.


#### Path Conditions
1. The current block height must be grater than the auction end block height hard-coded into the contract.
2. The first output box must hold the NFT and be owned by the pub key stored in R4.
3. The second output box must hold all of the Ergs and be owned by the auction holder.


#### Leads To
Exit

---



# How Informal Multi-Contract Specs Are Written

As you can see above, the stages of the whole contract are laid out separately, each with their own paths for how to spend them (perform an action). These paths then further specify the expected inputs/outputs together with the contract logic required (path conditions). Lastly, each path specifies whether it leads to another stage (which is hyperlinked) or if it is an exit path (meaning that the contract has completed and the tokens/data can move out of the multi-stage contract).

Clearly writing out the inputs + outputs as well as the path conditions can feel a little verbose, however this is on purpose. One only needs to look at the required inputs + outputs when writing off-chain code setting up the transaction for a given path. On the flip side, when writing code which will live on-chain (the contracts), one only needs to pay attention to the path conditions of a given stage.

Every stage, path, and the contract itself also has room for a preamble/explanation of what is going on in the given section. This allows for the spec writer to add extra comments and/or make clarifications so that it is painfully clear how the multi-stage contract is supposed to work when sharing with others.

Using markdown we get the benefit of being able to hyperlink the stages in the ToC, the paths for each stage, and where each exit path leads to. This provides us with a reasonable interface for going through informal specifications and understanding how the different stages are inter-linked. With both an eagle's eye view using the ToC as well as with a targeted/sequential view via following path hyperlinks, we have the ability to dig into how the parts and the whole of any given informal specification work.

With all of that said, let's look at the markdown of these informal specs to understand how they are written (also can be found [here in it's own file](./Multi-Stage_Spec_Template.md)).

```md
# Contract Name
Contract preamble

## ToC
1. [Bootstrap Stage](<#Bootstrap-Stage>)
2. [X Stage](<#X-Stage>)

## Bootstrap Stage
Preamble of how the contract is bootstrapped. There are no hard-coded values, registers,
or path conditions because this is the bootstrap stage and thus there is no contract yet.

### List Of Paths
- [Y Path](<#Y-Path>)
 
### Y Path
Preamble

#### Inputs
1. ...
2. ...
3. ...


#### Outputs
1. ...
2. ...
3. ...


#### Leads To
[X Stage](<#X-Stage>)

---

## X Stage
Preamble

### Hard-coded Values
- ...

### Registers
- R4: ...
- R5: ...


### List Of Paths
- [Y Path](<#Y-Path>)
 
### Y Path
Preamble

#### Inputs
1. ...
2. ...
3. ...


#### Outputs
1. ...
2. ...
3. ...


#### Path Conditions
1. ...
2. ...


#### Leads To
`Hyperlinked name of next stage` OR `Exit`

---

```

Alternatively if the list format above is too crowded for inputs/outputs/conditions, the following format is available as well for more complicated contracts which have a lot of registers and/or tokens.

```md
#### Inputs 
##### Input #1
...
##### Input #2
...

#### Outputs
##### Ouput #1
...
##### Ouput #2
...

#### Path Conditions
##### Condition 1
...
##### Condition 2
...

```

This allows you to use lists within each given input/output/condition section, which can be used for explaining logic related to registers for example.


# Conclusion

This is a first attempt at creating an easy-to-use informal scheme for specifying multi-stage contracts on the Ergo blockchain. It is not even close to perfect and likely can have a number of improvements in readability and traversability.

That said, there is great utility in using a scheme like this. We currently do have a great alternative that is more formal and visual called [FlowCards](https://github.com/ergoplatform/ergo-appkit/blob/design-contracts/docs/design-contracts/design-contracts.md), however it is considerably more complex to get started using compared to writing a spec in plain english. 

The markdown-based schema outlined in this document provides the writer the freedom to easily change what they've written as if they were writing an essay rather than the clunky feeling of writing compiled code (or a more formal spec). This flexibility is useful when one is working on figuring out how to implement a protocol and has the need to move fast/make edits quickly as previous ideas become obsolete/are found to be broken. Once an informal spec has been finalized, it can be a great idea to more formally specify it using FlowCards and gain the benefit of visualization (and potentially use it as an implementation tool if FlowCards eventually can compile down to ErgoScript).

If you have any thoughts/ideas for how to improve on the current scheme please feel free to make a suggestion.