Urbit: The Blockchain State Channel Solution No One Expected
==============================

For the uninitiated, state channels are the concept of  performing protocols by interacting with other users of a blockchain without actually using the blockchain itself for every action. To be precise, the blockchain is not used during the execution of the protocol, but only for opening and closing the state channel (technically your state channel can have multiple phases which may require more txs to snapshot the state at a given point in time, but that is besides the point).

The most popular form of state channel that you may have heard of is the Lightning Network on top of Bitcoin. The LN is a specific type of state channel called a payment channel. 

The LN is a great piece of innovation, however we are going to focus on state channels that deal with computation/smart contracts in this piece instead. Thus, let's take a cursory look at how state channels work.

One of the novel facets of users owning cryptographic keypairs is that arbitrary trustless protocols can be executed between 2 (or more) who are online at the same time. Every time a user performs an action, they can sign said action with their private key, guaranteeing they were in fact the one who authored it. All actions performed in the state channel can be validated by all other actors and then further signed off by them. This thereby validates an action in the state channel's history and thereby makes it part of the locally agreed upon state. In this scope for our use case, blockchains are treated as a primitive that allows for globally agreed upon state to be iteratively updated in checkpoint-style chunks based off of the closing results of these locally agreed upon states.

When compared to most protocols on the internet today, state channels require a very different design approach. In state channels we only have a single source of truth for the life of the channel, that being the state instantiated upon the opening of the state channel on-chain. On the internet today we rely on servers to provide a non-stop source of truth which makes it really quite easy to design a lot of protocols. With blockchains and state channels on the other hand, we must incorporate game theory, cryptography, and clever use of algorithms to make sure that the protocol works as intended and bad actors cannot game the system.


---
Encoding Protocols Via State Channels
---

Given that we only have a single source of truth (the blockchain) which is inherently very costly (compared to running servers), we need to build protocols that begin by running a deterministic "instantiation" function. This function takes data which was retrieved from the blockchain upon state channel opening as input. In essence the blockchain is providing the environment (the global state which the protocol refers to when beginning), and for most protocols, a pseudo-random seed (a piece of data which acts as a source of randomness for the deterministic instantiation of the current channel).

The ruleset of the protocol which all actors follow is completely off-chain and must be agreed-upon by everyone involved rather than by a centralized server. What this usually means in practice is that everyone runs the same client and so the results of each person's actions can be understood and signed off on by all other participants in the protocol as a valid action. A client acts as both a front-end for an actor to participate in the protocol, as well as a validator that any given action submitted by other participants is in fact acceptable according to the ruleset.

To make this a bit more understandable, let's bring in a naive example via the game of Go. Two players agree to play Go using a smart contract on a blockchain to keep track of the wins/losses and to choose who gets white and who gets black (optionally also payout the winner if these are money matches). They initiate the state channel on chain by issuing a transaction to said smart contract with each person signing off that they are agreeing to play a game. Once the transaction is processed, the results of the smart contract computation (a pseudo-random seed) is then used by both players' clients to decide who gets which colour. 

With a 2 person game such as Go it is trivial to decided on an algorithm for how the seed could be used to decide player colour. An arbitrary example: The player with the address who's value is lower could be assigned "odd" while the player with the higher value address is assigned "even". If the pseudo-randomly generated seed is odd, then the prior player gets assigned white, while if it's even, then the latter.

Now both players connect to each other peer2peer using their clients and start a match which refers to the state channel initiation transaction on-chain. Each person's client reads the seed from the blockchain, runs it through the color choosing algorithm, and sets up the local game state for each player with their respective colors.

Every time a player performs a move they sign off on it themselves and then send the signed action to the other player. The other player's client verifies that the action is valid, signs off on it, and then sends it back to the original player who performed the move. Now both players have co-operatively agreed to move their local state 1 move forward. This continues to go on and thus a history of all the moves is created and acts as a proof of what happened.

The players continue to play their whole game move-by-move until one person wins. Both players sign off on the results of the match which then get posted on-chain and are computed/verified within the smart contract, thereby increasing each player's win/loss count (and releasing funds to the winner if it was a money match).

To be clear, the example above is overly simplistic, inefficient, and actually quite abusable if one person decides to stop playing, but that is besides the point for this current post. This is a naive example to demonstrate how state channels work in general.


---
Why State Channels Suck Today And How Urbit Is The Solution
---

State channels are a great idea, but one of the main problems is that there is a distinct lack of a messaging layer which easily allows participants to communicate to perform said state channels. Nearly everything out there today is centralized and has no real ability to integrate cleanly with blockchains such as Ethereum.

Urbit appears to be the perfect solution here. Not only does it provide direct p2p communication based on cryptographic identities that can be tied to Ethereum addresses, it also provides a computation platform to run these state channels on. 

Let's break that down a little. Since users can own Azimuth points within their Ethereum wallet on-chain, this means that a user's Urbit identity is already directly linked to their Ethereum identity/account. Therefore if a user with an Urbit ID NFT on Ethereum initiates a state channel, all other participants already know exactly how to communicate with said user (using their @p).

Furthermore Urbit already has some Ethereum integration built in (from what I understand) which means that adding support for deeper integration for state channels shouldn't be that excessively challenging. This means that state channel landscape apps should be possible which merely ask the user to input the id of the initiation tx from the Ethereum mainnet, and then the app itself pulls everyone else's @p's from looking at the Urbit NFTs held at the other accounts who are taking part in the protocol. From there all actors' ships can communicate p2p, and using the client code in the app, perform the state channel protocol. 

With our naive Go example, both players would initiate the state channel on-chain, copy the tx id, and paste it into their Go Urbit landscape app. The app will find the opponent's @p automatically, instantiate the connection, and both players will find each other with a goban GUI ready to play a game of Go. They each take turns playing moves, and when the result is finished a closing tx is generated by the app and is posted on-chain to close the state channel.

It is even possible that these state channels could be first agreed upon on Urbit. A user creates an Urbit group for a state channel protocol they wish to run. They open up the state channel app and select said group to run the protocol with. All members of the group are sent invitations to join the protocol, which requires that they provide a signature from their Eth keypair to take part. Once all members send their signed agreement to join, the "host" then posts the tx on-chain to start the state channel (if others are required to submit funds, then it could require txs from everyone). Once the state channel tx is confirmed then the host provides all of the ships in the group the tx id. Their clients then use the generated data to initiate the protocol locally, and since all the ships are already communicating, the protocol moves forward effortlessly.

---
Conclusion
---

As can be seen, Urbit very naturally fits as if it was designed to do so in the first place.

This means that Urbit can theoretically be a part in aiding blockchains like Ethereum to scale by providing an identity/messaging/computation layer to run state channels on. On the flip side, this kind of scheme provides Urbit users the ability to run more complicated p2p protocols which are inherently low-trust/rely on distribution of funds based on events that occur without needing to trust a 3rd-party/entity.

Whether all of this will be looked into/implemented at all, only time will tell. Nonetheless I believe there is a decent amount of potential here which hasn't been tapped into yet by any other project I've seen.

If anyone has any thoughts/comments on the topic I'd be happy to hear from you.

~mocrux-nomdep