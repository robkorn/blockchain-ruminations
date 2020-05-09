Efficient Global Context Claims & Accumulators In Blockchains
===
Recently I have been working through designing various DeFi protocols in the Ergo contract model, and the thought came to mind that to make things more efficient it would be of great benefit if we could make claims about the global state of the blockchain/UTXO-set. This lead me to come up with the concept of global data accumulation (which I will explain in a moment) however I didn't realize that the concept is even further extendable to any claim about the blockchain state until a conversion with @jasondavies and @scalahub where they pointed out negative proofs are possible.

So, what are these "Efficient Global Context Claims" then? In essence, smart contract enabled blockchains to date have various levels of context available to developers in the smart contracts themselves. Some may allow little to no outside data from other accounts/utxos/contracts (Algorand or Cardano), while others like Ethereum only allow smart contracts to specifically call one anothers' functions to return a value and obtain that tiny slice of context that way. What all of these systems/models lack is the ability to make claims about the state of every single account/UTXO/smart contract on the entire system.

There are clear reasons for this, it is horribly inefficient. With blockchains like Ethereum where it's almost impossible for a user to bootstrap a full node, expecting a validator to go through the entire state of the blockchain to validate a logical check in a single contract is **impossible** to even consider implementing when they're already constantly hitting limits. Ergo on the other hand is the only blockchain out there which has smart contracts on a UTXO-based system *and* has the concept of data-inputs.

For those who are not familiar, data-inputs allow you to reference a UTXO/box in a transaction without spending it. Thus you have access to all of the data/value of tokens within a UTXO without needing to have the right to spend it. Following that line of reasoning, this means we can check data from any dApp/contract on the blockchain without needing permission or having to encode anything at all in the contract to support doing so. This has major implications which we will now get into. 
Because data-inputs allow anyone to reference the registers and tokens of any box, this means that we can make generic claims about the whole system and have any user on the system prove/disprove said claims. For example we can arbitrarily create a smart contract that specifies `This UTXO can only be spent if another UTXO exists with the number 5 in Register R4 and it holds 5 Erg`. If such a box exists with 5 Erg and the number 5 in register R4, then anyone can create a transaction using said box as a data-input, thereby proving that such a box exists and giving them the right to spend the UTXO with out smart contract.

Now, what if we were to include a *reward* (say 100 Ergs) together with our smart contract which is making a claim about the global context? This would provide a monetary incentive for actors (who are not the validators) to scan the entire UTXO-set to find a box that proves the claim at hand, thereby earning the actor money. Thus we could create a box which holds 100 Ergs and specifies the following claim in it's smart contract: `This UTXO can only be spent if a box exists which holds at least 1000 Ergs at the smart contract address "2iHkR7CWvD1R4j1yZg5bke..."`. If such a box exists, then anyone can cite the box as a data-input and earn the 100 Ergs. This is a trivial task, and would earn someone about $20.00 USD (in today's Erg/USD conversion rate).

Thus, if our smart contract box does not get spent over the course of the proceeding blocks, we come to the conclusion that such a box does not exist on the network. Thus we have outsourced the scanning and verifying of the global state of the blockchain to users who have a monetary incentive, while making it seamless and trustless due to using data-inputs. 

Now, if we implement software to streamline this task automatically and make it trivial for Ergo users to run at a profit, we can gain a high level of assurance about the claims we make on-chain (the greater the reward the greater the incentive for "verifiers" to make sure that the claim does/does not hold thereby increasing the assurance). One can view this as an efficient and trustless (due to data-inputs providing proof of validity) variation of oracles where each oracle only processes data from the blockchain and folds it down into a boolean result. This should be possible to run on commodity hardware by anyone anywhere in the world with an internet connection and computer, allowing for a truly decentralized system of "verifiers" who make it possible to make efficient global context claims about the entire blockchain. 

This also acts as a superb way for users to come into the ecosystem and earn Erg while providing a truly pivotal service with a low barrier of entry. Once the utxo-set grows very large and the claims become more complex, we could even imagine "verification pools" coming about, where users are given a segment of the UTXO-set to keep track of/process and receive a small portion of any rewards that their "verification pool" wins. This opens up a lot of doors of opportunity.

With what I've outlined above, we have the ability to make specific positive and negative claims about the global state of the blockchain and every token and contract on it. Very powerful. However, we can do even more which I alluded to in the beginning.

#### Global Context Data Accumulators

Since data-inputs provide us with full context of a given UTXO, it means that we can also apply arbitrary functions to said data instead of just boolean checks. Thus, we can have our simple "verifiers" actually become "accumulators" who scan the entire blockchain to find all relevant boxes, and then fold the data down into a single/set of values which get posted into a new single box.

For example, let's say we want to know the total number of Erg which are held in a smart contract at the address of `2iHkR7CWvD1R4j1yZg5bke`.

We can post a box with a 100 Erg reward on-chain with a contract that specifies anyone can spend the box if:
1. The user provides at least 1 data-input of a box owned by `2iHkR7CWvD1R4j1yZg5bke`.
2. The user creates a single new box with:
    - The number of data-inputs he provided in register R4
    - The accumulated data in R5 (total number of Ergs held in all data-inputs)
    - His public key in register R6
    - This box locked under the "Finalizing Accumulation" script


This new box under the "Finalizing Accumulation" script finalizes/ends the accumulation if no one spends it in the next 5 blocks. However it allows anyone to spend the box if they:
1. Provide at least 1 more data-input than the value in register R4.
2. Create a single new box with:
    - The number of data-inputs he provided in register R4 
    - The accumulated data in R5 (total number of Ergs held in all data-inputs)
    - His public key in register R6
    - This box locked under the "Finalizing Accumulation" script again

Thus, the user has proven that he/she has found more data-inputs than the previous user which match the stated restrictions. Once the accumulation has finalized because no one has spent it after 5 blocks, the user who has their address in R6 can withdraw his reward and move the value in R5 (the accumulated datapoint) into a new box of it's own in a register allowing anyone on the entire network to easily reference it as a data-input.

Admittedly this is a slightly naive way to implement an accumulator scheme (as it could take arbitrarily long for the datapoint to finalize, and probably could use an NFT to make it easy to find the finalized datapoint for example), but it was chosen to be as clear as possible for how accumulators could be implemented/work. We can likely build out "data accumulation pools" where there are hard-capped time limits for protocols that require consistent data accumulation.

Context data accumulators allow us to sum up the state of the blockchain into one or more registers of a new box. This can be very useful in many use cases such as in DeFi where you may need to know the total funds held in a liquidity pool contract in order to calculate interest rates accurately. Using context data accumulators to checkpoint/accumulate the state of the blockchain allows users to parallelize actions within dApps inside of their own UTXOs completely separate from others, while still contributing to a global shared core state of the dApp that is put together by the accumulators. It may even be possible to one day merge such schemes with sharding and/or side-chains to allow for scaling of smart contracts thanks to the nature of the UTXO model and this snapshot/accumulation approach to contract design.


#### Conclusion

With all of that said, thanks to Ergo's UTXO model with data-inputs, we have the ability to trustlessly fold the whole UTXO-set state down into a single value. By leveraging monetary incentives, we can thus use this to create efficient claims on global blockchain context and/or accumulate data into a single box that is readable by anyone and everyone.

There is likely a lot more to be said/thought about with all of this, but I believe this should be good enough of a starting point to get the idea out there in an understandable form.
