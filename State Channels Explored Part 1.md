State Channels Explored Part 1: How To Encode Basic Protocols
=================================

For the uninitiated, state channels are the concept of interacting with other users of a blockchain via the use of your cryptographic keypairs, however without actually using the blockchain itself. To be precise, the blockchain is not used at all, except for opening and closing of the state channel.

The most popular form of state channel currently out there today is most definitely the Lightning Network on top of Bitcoin. The LN is a specific type of state channel called a payment channel. The LN has tons of utility and is a great idea, however it is a tired topic and there are much more interesting problems to be tackled with state channels (in my opinion).

One of the novel facets of having cryptographic keypairs available is that arbitrary protocols can be produced between 2 (or more) peers who are online at the same time. Every time an actor performs an action, they can sign said action with their private key, guaranteeing they were in fact the one who authored it. Every action can also be validated by all other actors in the state channel, and then signed off thereby making it part of "locally agreed upon state". In this scope, blockchains are in essence a primitive that allows for "globally agreed upon state" to be iteratively updated in checkpoint-style chunks based off of the closing results of these "locally agreed upon states".

This is all likely obvious to most of you reading this, but with these basic tools it really is possible to encode a wide variety of protocols. The key difference however compared to most protocols on the internet today is that within a state channel we only have a single source of truth for the life of the channel, that being the state instantiated upon the opening of the state channel on-chain. On the internet today we rely on servers to provide a non-stop source of truth which makes it really quite easy to design a lot of protocols. With blockchains and state channels on the other hand we must incorporate game theory, cryptography, and clever use of algorithms to make sure that the protocol works as intended and bad actors cannot game the system.


---
Encoding Protocols Via State Channels
---

Given that we only have a single source of truth which is inherently very costly (compared to running servers), we need to build protocols that begin by running a deterministic "instantiation" function which takes as input data which was retrieved from the blockchain upon state channel opening. In essence the blockchain is providing the environment (the global state which the protocol refers to), and for most protocols, a pseudo-random seed (a piece of data which acts as a source of randomness for the deterministic instantiation of the current channel).

The ruleset of the protocol which all actors follow is completely off-chain and must be agreed-upon by everyone involved rather than by a centralized server. What this usually means in practice is that everyone runs the same client and so the results of each person's actions can be understood and signed off on by all other participants in the protocol as a valid action. A client acts as both a front-end for an actor to participate in the protocol, as well as a validator that any given action submitted by other participants is in fact acceptable according to the ruleset.

To make this a bit more understandable, let's bring in an example via the game of Go. Two players agree to play Go using a smart contract on a blockchain to keep track of the wins/losses and to choose who gets white and who gets black. They initiate the state channel on chain by issuing a transaction to said smart contract with each person signing off that they are agreeing to play a game. Once the transaction is processed, the results of the smart contract computation (a pseudo-random seed) is then used by both players' clients to decide who gets which colour. 

With a 2 person game such as Go it is trivial to decided on an algorithm for how the seed could be used to decide player colour. An arbitrary example: The player with the address who's value is lower could be assigned "odd" while the player with the higher value address is assigned "even". If the pseudo-randomly generated seed is odd, then the prior player gets assigned white, while if it's even, then the latter.

Now both players connect to each other peer2peer using their clients and start a match which refers to the state channel initiation transaction on-chain. Each person's client reads the seed from the blockchain, runs it through the color choosing algorithm, and sets up the local game state for each player with their respective colors.

Every time a player performs a move they sign off on it themselves and then send the signed action to the other player. The other player's client verifies that the action is valid, signs off on it, and then sends it back to the original player who performed the move. Now both players have co-operatively agreed to move their local state 1 move forward. As such, a history of all the moves is created and acts as a proof of what happened.

The player's continue to play their whole game move-by-move until one person wins. Both players sign off on the results of the match which then get posted on-chain and are computed/verified within the smart contract, thereby increasing each player's win/loss count.

The example above is overly simplistic, inefficient/optimizable, and actually quite abusable if one person decides to stop playing, but that is besides the point for this current post and so we'll cover that another day.


---
Conclusion
---

There genuinely is a lot that's possible with the basic model explained above. That said the challenge with encoding actual complex protocols with state channels is that we must somehow make them fit. In short it requires carefully defining a strict ruleset which allows all participants to be able to, and furthermore be incentivized, to validate that other participants are acting properly.

All random events must be based off of the original seed, the results of the actions of the participants, or both together. These pieces of data must be run through a deterministic (pure) function which spits out a "random" value to then be used by all players. Every part of the protocol which requires randomness that can not be gamed must utilize opposing incentives of participants along with randomness effected by player actions.

And this only covers the fairness of the inter-protocol actions which are to be accounted for. There are also meta-incentives (incentives outside of the single instantiation/state channel but instead the stability of the protocol in relation to the global state held on the blockchain) which make designing these protocols even more challenging.

In the next episode we'll get our first taste of the actual fun stuff: Examining Meta-Incentives And The Reward Problem.