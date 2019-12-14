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

As we have an interface definition, there are a number of great benefits we can take advantage of. The four key being:

1. Automatic generation of pre-conditions for formal verification based on allowed input.
2. Automatic generation of an input validator.
3. Automatic generation of dApp Graphical User Interface which the end-user can interact with.
4. Providing a standard which allows for front-end/dApp development frameworks to build on top of.


Packaging The dApp To Achieve Wide Usage
----------

Furthermore we can package all of the data/files together into a pre-defined package format (say with `.dApp` extension). This then allows for a dApp to be easily utilized by any and all `.dApp` supporting tooling.

In the future wallets could support the importing of `.dApp` packages and automatically generate GUIs right in the wallet itself that users utilize to interact with the dApp. This makes it considerably easier for dApps to be tested, shared, and thus decentralizing the front-end of dApps (which often rely on webpages and/or only provide a single front-end period).

In the realm of creating polished custom-made dApp front-ends, there is also a lot to gain as well. If a developer obtains a `.dApp`, down the line they could use a library/framework which plugs right in and allows them to use css/html/js to specify the design & the flow of the GUI manually. The developer is provided the guarantee that all inputs are valid before submitting the transaction while also having a lot of the boilerplate auto-generated thus allowing the front-end implementor to focus on what matters.


The Interface Definition
----------

The interface definition is created via an Interface Description Language ([IDL](https://en.wikipedia.org/wiki/Interface_description_language)).

The IDL needs to support specifying the exact inputs a user is allowed/required to provide. This IDL could be a brand new language which offers more power (and generality) at the cost of requiring parsers to be written, or could be an eDSL (embedded Domain Specific Language) implemented within a language that is the go-to in the ecosystem.

We will now proceed to examine examples (in pseudo-code) of what would be required in defining interface definitions with an IDL to be able to achieve all of the automated benefits listed above.

Defining Valid Inputs
----------

At the core, the interface definition relies on specifying pre-conditions. In other words, we are defining which inputs are allowed to be submitted to our contract. Ergo the name "interface definition".

Having pre-conditions defined for your dApp has great benefits in that they map directly onto pre-conditions used in automated formal verification (which allow for shrinking the statespace to only the valid inputs that are possible).

The below pseudo-code IDL example displays one way which this can be encoded.
```
   let amountToSend = Int.range(0,1500);

   let stringChoice = String.oneOf(["Yes", "No", "Maybe"]);

   let howMany = Int.oneOf([2, 4, 6, 8]);

   let recipientPubkey = String.ofLength(35);
```

This kind of IDL should be trivial to encode within any modern language. These functions shown above can simply evaluate to boolean functions which take in the expected value as input.

For more complex contracts, the IDL may need to be more powerful and allow the chaining of functions for defining pre-conditions.
```
    let goodString = String.ofLength(18).noElement("x");
```

Furthermore, there may situations where a pre-condition needs to rely upon the the value of another input. Ideally the IDL would work as such:

```

    let favoriteNumber = Int.range(0, 100);

    let choicesToChooseFrom = Int.oneOf([favoriteNumber, 25, 50, 100]);
    
    let favoriteRange = Int.range(0, favoriteNumber);

```

In such cases, the implementation of this begins to get more complex because `favoriteNumber` in one context means the boolean function itself, and in another it means the actual input value which has successfully passed the pre-condition.

Besides the parsing angle however, the resulting code generated by a scheme like this isn't that complicated as we will see in the next section.

Automatic Generation Of Input Validator
------------

Going forward, we would like to be able to generate an input validator based off of our interface definition. At it's simplest, this is a function which takes all the expected inputs, and runs them against our defined pre-conditions making sure that they are indeed valid (ie. value in range, value is one of the specified values, etc.). 

For most languages, this is most easily encoded via the use of a datatype/struct.

The datatype's constructor is specified to accept all the expected inputs (on the type level), and then encodes checks to verify that all of the inputs then match the pre-conditions specified. The constructor can return an Optional value of either the successfully created datatype, or an Error specifying which of the specific pre-conditions failed (and the corresponding input). In this way, if the datatype succeeds in being constructed, the developer has the guarantee (based off of the interface definition spec) that the inputs are all valid.

However there is one key thing missing in our interface definition to allow for this to work. We have defined all of the required inputs possible for the given smart contract, however we have not considered the fact that only a sub-set of these inputs may be required for any given _action_ which a participant in a smart contract can perform.

As such we have to define said actions, which in essence are simply a grouping of pre-conditions put together. (The astute among you will realize this also clearly maps onto structs/datatypes.) 

A dApp will often have more than one action, but we will keep things simple and explore what defining a single one would look like.

IDL pseudocode:
```
    let lockAmount = Int.range(1,10);
    let lockPassword = String.maxSize(35);
    let betterLockPassword = String.minSize(lockPassword.length() + 5);

    action lockFunds(lockAmount, lockPassword);
```

What this means is that an action called `lockFunds` is defined as a valid action for the smart contract which takes 3 input values. Each input must be of the correct base type (ie. Int, String), and must also pass the defined pre-condition in order for it to be valid.

This IDL can then be transformed into a struct/datatype such as the below pseudo-code:

```
    struct LockFundsAction = {
        lockAmount : Int,
        lockPassword: String,
        betterLockPassword: String,
    }

    impl LockFundsAction {
        fn new (lockAmount: Int, lockPassword: String, betterLockPassword: String) -> Optional<LockFundsAction> {
            let isValidLockAmount = lockAmount >= 1 && lockAmount <= 10;
            let isValidLockPassword = lockPassword.length <= 35;
            let isValidBetterLockPassword = betterLockPassword.length >= (lockPassword.length + 5)
            if (!isValidLockAmount) {
                return NotInRangeError(lockAmount);
            }
            else if (!isValidLockPassword) {
                return SizeTooLarge(lockPassword);
            }
            else if (!isValidBetterLockPassword) {
                return SizeTooSmall(betterLockPassword);
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

Thus we now have an input validator created via meta-programming based off of our IDL.

_Note: From the implementor's perspective of this more advanced IDL, the pre-conditions must now pass on their core type requirement as well as the boolean function. In languages that have first-class types, you could just make the results from the pre-condition generation functions be a tuple which is made of (type, boolean function) for example. However as this isn't the case for most languages, there may need to be a bit more work done behind-the-scenes in your given implementation language to have type inference work from the boolean function._


Automatic GUI Generation
-------------

Now with an action struct created, it is quite trivial to automatically generate GUI elements which take in values of the corresponding type (Int, String, ...).

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
