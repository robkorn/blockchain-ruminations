Universal dApp Interface & Package Standard
=================================

Motivation 
----------

One of the key issues in our time in development of dApps is the fact that after the smart contract is written, it is inherently challenging to interact with and provide a full-fledged UI for an end-user. The idea of having multiple user interfaces for a single smart contract/dApp seems laughable since as of the current moment there are no standards set at all.

As there are no standards, we have no ability to create generalized frameworks or tooling around dApps which allow people to reuse the work of others or allow a dApp to be a modular entity that can be imported and used in different environments.

Instead with the current situation we find ourselves in we simply throw smart contracts on-chain and have every single project create custom interfaces/tools to interact with the smart contract. We have no concept of defining an interface for the off-chain portion of the dApp (which interacts with the on-chain code) that is free from the implementation of the code itself.

Thus we are forever chained to an endless myriad of custom made user interfaces (often which tend to be centralized on a website to boot) which are not reusable and impair the ability for dApps to be easily distributed/accessed, or have actual longevity past the lifespan of a given implementation.


Rethinking What A dApp Is
----------

With the points mentioned above, I ask the reader to take a step back and consider that we may need to rethink about how we define a dApp.

Instead of considering a dApp to be the combination of a front-end implementation together with a smart contract, what if instead we considered what constitutes a dApp to be the combination of:

1. A smart contract (or multiple for the given dApp)
2. An interface definition
3. Miscellaneous contextual data (dApp name/description via a config file, image icons, formal specs/proofs, ...)

Thus instead of a dApp just being a given ephemeral instantiation which end-users can interact with, it is principally the smart contract + the specification of the interface for how to interact with the smart contract (with extra useful data/files that provide better assurance or enrich the end-users experience)

The difference here is key, in that the specification is reusable and does not need to be tied down to a specific language, front-end, or anything else. 


The Benefits Of This New Definition Of A dApp
----------

What this unlocks for us is the ability to have the actual core logic of the dApp be specified in a generalized, machine understandable, and reusable way.

As we have an interface definition, there are a number of great benefits we can take advantage of. The three key being:

1. Automatic generation of pre-conditions for formal verification based on allowed input (thereby shrinking state space to only what is relevant)
2. Automatic generation of input validator.
3. Automatic generation of dApp Graphical User Interface which the end-user can interact with.


Packaging The dApp To Achieve Wide Usage
----------

Furthermore we can package all of the data/files together into a pre-defined package format (say with `.dApp` extension). This then allows for a dApp to be easily utilized by any and all `.dApp` supporting tooling.

In the future wallets could support the importing of `.dApp` packages and automatically generate GUIs right in the wallet itself that users utilize to interact with the dApp. This makes it considerably easier for dApps to be tested, shared, and thus decentralizing the front-end of dApps (which often rely on webpages and/or only provide a single front-end period).

In the realm of creating polished custom-made dApp front-ends, there is also a lot to gain as well. If a developer obtains a `.dApp`, down the line they could use a library/framework which plugs right in and allows them to use css/html/js to specify the design & the flow of the GUI manually. The developer is provided the guarantee that all inputs are valid before submitting the transaction while also having a lot of the boilerplate auto-generated thus allowing the front-end implementor to focus on what matters.


The Interface Definition
----------

The interface definition is created via an Interface Description Language ([IDL](https://en.wikipedia.org/wiki/Interface_description_language)).

Optimally the used IDL supports specifying the exact inputs a user is required to provide to be able to design rich & specific interfaces. This IDL could be a brand new language which offers more power (and generality) at the cost of requiring parsers to be written, or could be an eDSL (embedded Domain Specific Language) implemented within a language that is the go-to in the ecosystem.

We will now proceed to examine examples (in pseudo-code) of what would be required in defining interface definitions with said IDL to gain the previous stated benefits.

Defining Valid Inputs
----------

At the core, the interface definition is made


IDL Pseudo-example:
```
   let amountToSend = Int.range(0,1500);

   let stringChoice = String.oneOf(["Yes", "No", "Maybe"]);

   let howMany = Int.oneOf([2, 4, 6, 8]);

   let recipientPubkey = String.ofLength(35);
```

This kind of IDL should be trivial to encode within any modern language. These functions shown above can simply evaluate to boolean functions which take in the expected value as input.

Automatic Generation Of Input Validator
------------

Next, we would like to be able to generate an input validator based off of our interface definition. At it's simplest, this is a function which takes all the expected inputs, and validates that they match the interface definition (ie. value in range, value is one of the specified values, etc.). 

For most languages, this is most easily created via the use of a datatype/struct.

The datatype's constructor is specified to accept all the expected inputs (on the type level), and then encodes checks to verify that all of the inputs then match the pre-conditions specified. The constructor can return an Optional value of either the successfully created datatype, or an Error specifying which of the specific pre-conditions failed (and the corresponding input). In this way, if the datatype succeeds in being constructed, the developer has the guarantee (based off of the interface definition spec.) that the inputs are all valid.

However there is one key thing missing in our interface definition to allow for this to work. We have defined all the required inputs possible for a given smart contract, however we have not considered the fact that for a given _action_ which a participant in a smart contract can perform, only a sub-set of these inputs may be required.

As such we have to define actions, which in essence are simply a group of pre-conditions put together.

IDL pseudocode:
```
    let lockAmount = Int.range(1,10);
    let lockPassword = String.maxSize(35);

    action lockFunds(lockAmount, lockPassword);
```

What this means is that an action called `lockFunds` is defined as a valid action for the smart contract which takes 2 input values. Each input must be of the correct base type (ie. Int, String), and must also pass the pre-condition.

This IDL can then be transformed into a struct/datatype such as this example pseudocode:

```
    struct LockFundsAction = {
        lockAmount : Int,
        lockPassword: String,
    }

    impl LockFundsAction {
        fn new (lockAmount: Int, lockPassword: String) -> Optional<LockFundsAction> {
            let isValidLockAmount = lockAmount >= 1 && lockAmount <= 10;
            let isValidLockPassword = lockPassword.length <= 35;
            if (!isValidLockAmount) {
                return NotInRangeError(lockAmount);
            }
            else if (!isValidLockPassword) {
                return SizeTooLarge(lockPassword);
            }
            else {
                return LockFundsAction {
                    lockAmount: lockAmount,
                    lockPassword: lockPassword,
                }
            }
        }
    }

```

And as such an input validator can be created via metaprogramming to great benefit.

Note: From the implementor's perspective of this more advanced IDL, the pre-conditions must now somehow be able to pass on their core type requirement as well as the boolean statement. In languages that have first-class types, you could just make the results from the pre-condition generation functions be a tuple which is made of (type, boolean function). As this isn't the case for most languages, there may need to be more work done behind-the-scenes to someone do type inference from the boolean function.


Automatic GUI Generation
-------------

Now with this action struct created, it is quite trivial to automatically generate GUI elements which take in values of the corresponding type (Int, String, ...).

Potentially expanding on the the IDL by allowing developers to specify ideal GUI elements for an input could be useful.

Example pseudocode:
```
    # lockAmount.GUI: Slider
    let lockAmount = Int.range(1,10);

    # lockPassword.GUI: InputBox
    let lockPassword = String.maxSize(35);

```

However there is one final piece missing with our IDL up to this point. We can generate GUIs automatically and validate the user's input from the GUI, but we haven't yet specified how the values are to be used in performing the actual action (submitting the transaction with the given inputs).

Action Submission
-------------

Here we specify the specifics of how a transaction is created using the inputs.


Here is a pseudo-example of how this could work in the IDL:

```
    # lockAmount.GUI: Slider
    let lockAmount = Int.range(1,10);
    # lockPassword.GUI: InputBox
    let lockPassword = String.maxSize(35);

    action lockFunds(lockAmount, lockPassword) {
        return ({
                smartContract = dApp.context.smartContract,
                register.R4 = lockPassword,
                amount = lockAmount
            })
    }


```
_(As can be seen, I also included the idea of importing the smart contract which has been included within the same `.dApp` that this interface definiton is in.)_

This then can be translated into a new method for creating the action's transaction on our `lockFundsAction` struct:

```
    impl LockFundsAction {
        fn createTransaction (self) {
            let tx = createAndSignTransactionWithData({
                        smartContract = dApp.context.smartContract,
                        register.R4 = self.lockPassword,
                        amount = self.lockAmount});
            return(tx);
        }
    }

```



Going Forward
----------

This document is primarily a high-level overview of the concept of a Universal dApp Interface & Package Standard, rather than a specific definition of one. Chances are that each given platform in the blockchain space would likely have to create such a standard for themselves based on the requirements/most popular tooling in the ecosystem at hand. Nonetheless, it is my general contention that on the high-level, such a standard is one of the better ways forward for dApp development at large, even if each ecosystem has it's own customizations on the specifics.

A few of us in the Ergo community have already began to ruminate on how a Universal dApp Interface & Package Standard can be integrated within [AppKit](https://github.com/aslesarenko/ergo-appkit) to achieve `.dApp` support in the primary cross-language polyglot dApp library for the Ergo ecosystem, and also how it could be integrated with [Stainless](https://github.com/epfl-lara/stainless) to improve the formal verification process of smart contracts.
