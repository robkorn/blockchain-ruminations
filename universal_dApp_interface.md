Universal dApp Interface & Package Standard
=================================

Motivation 
----------

One of the key issues in our time in dApp development is the fact that after the smart contract is written, it is inherently challenging to interact with and provide a full-fledged UI for the end-user. The mere thought of having multiple user interfaces for a given contract/dApp seems like a laughable waste of time currently since there are no standards set to allow for it.

As there are no standards, we have no ability to create generalized frameworks or tooling around dApps which allow people to reuse the work of others or allow a dApp to be a modular entity that can be imported and used within different environments.

Instead we find ourselves simply throwing smart contracts on-chain, creating custom UIs for each one, and then ostensibly calling the combination of the two pieces a "dApp". 
Thus we are forever chained to an endless myriad of custom made user interfaces (often which tend to be centralized on a website) which are not reusable or inter-operable. 

This severely impairs the ability for dApps to be easily distributed/accessed, or have actual longevity past the lifespan of a given implementation. 
As such we are left spinning in circles with our present model of dApps.


Rethinking What A dApp Is
----------

With that said, I ask the reader to take a step back and consider that we may need to rethink about how we define a dApp.

Instead of defining a dApp as the combination of a front-end implementation together with a smart contract, what if instead we considered what constitutes a dApp to be the combination of:

1. A smart contract (or multiple for the given dApp)
2. An interface definition
3. Miscellaneous contextual data (dApp name/description via a config file, image icons, formal specs/proofs, ...)

Thus instead of a dApp just being a given ephemeral instantiation which end-users can interact with, it is principally the smart contract + the specification of the interface for how to interact with the smart contract (plus extra useful data/files that provide better assurance or enrich the end-users experience).

The difference here is key, in that the specification is reusable and will live on even if a specific front-end implementation becomes deprecated. 


The Benefits Of This New Definition Of A dApp
----------

What this unlocks for us is the ability to have the actual core logic of the dApp be specified in a generalized, machine understandable, and reusable way. 

Not only will our dApp exist past any single implementation, but because we have a dApp interface definition, we get rewarded with several benefits that are as of yet not possible:

1. Provides a standard which allows for front-end/dApp development frameworks to build on top of.
2. Automatic generation of pre-conditions for formal verification.
3. Guaranteed input validity when a user interacts with the dApp.
4. Automatic generation of a Graphical User Interface which the end-user can interact with.


Packaging The dApp To Achieve Wide Usage
----------

Furthermore, since we are abstracting out dApp out of any single instantiation by means of an interface definition, we can now also package our dApp in a generalizable and standardized way.
All of the data/files can be put together in a pre-defined package format (say with the `.dApp` extension) which allows for a dApp to be easily utilized by any and all `.dApp` supporting tooling.

In the future wallets could allow importing of `.dApp` packages which automatically generate GUIs right in the wallet itself that users utilize to interact with the dApp. This makes it considerably easier for dApps to be tested, shared, and thus decentralizing the front-end of dApps (which often rely on webpages and/or only provide a single front-end period that is tied to a specific wallet).

In addition, there is a lot to be gained in the realm of polished custom-made dApp front-ends from this setup as well. If a developer obtains a `.dApp`, they could use a library/framework which plugs right in and allows them to use css/html/js to specify the design & the flow of the GUI manually.
The developer also has the guarantee that all user-provided inputs are valid before submitting the transaction, thus simplifying the entire development process while enriching the experience with more safety.

Now to the interesting part, the interface definition itself.

The Interface Definition
----------

The interface definition is created via an Interface Description Language ([IDL](https://en.wikipedia.org/wiki/Interface_description_language)).

The IDL needs to support specifying the exact inputs a user is allowed/required to provide. This IDL could be a brand new language (which offers more power and generality) at the cost of requiring parsers to be written, or could be an eDSL (embedded Domain Specific Language) implemented within a language that is the go-to of a blockchain ecosystem.

We will now proceed to examine examples (in pseudo-code) of defining an interface definition using a (pseudo) IDL which achieves all of the benefits mentioned above.

Defining Valid Inputs
----------

At the core, the interface definition relies on specifying pre-conditions. In other words, we are defining which inputs are allowed to be submitted to our contract. Therefore the name "interface definition".

Having pre-conditions defined for your dApp is quite useful, as it forces you as the developer to really think about specifying the exact inputs allowed (thereby lowering the chance that weird edge-cases can happen due to using broad types like Int32).
Furthermore, these pre-conditions also directly map onto pre-conditions used in automated formal verification (which allow for shrinking the statespace to only the valid inputs that are possible).
And lastly, specifying pre-conditions is a requirement for all of the other fanciful features which we will get to later on.

The below pseudo-code example displays one way which pre-conditions could be encoded via our imaginary IDL.
```
let amountToSend = Int.range(0,1500);

let stringChoice = String.oneOf(["Yes", "No", "Maybe"]);

let howMany = Int.oneOf([2, 4, 6, 8]);

let recipientPubkey = String.ofLength(35);
```
As can be seen, this is quite simple to understand and should be trivial to implement within any modern language.
These functions used above can simply evaluate to boolean functions which take in the expected value as input and return true/false whether the input is valid.

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

Putting aside the parsing perspective however, the resulting code generated by a scheme like this isn't that complicated as we will see in the next section.

Automatic Generation Of Input Validator
------------

Going forward, we would like to be able to generate an input validator based off of our interface definition. At it's simplest this is a function which takes all the expected inputs and runs them against our pre-conditions making sure that they are indeed valid (ie. value is in range, value is one of the specified values, etc.). 

For most languages, this is most easily encoded via the use of a datatype/struct.

The datatype's constructor can be made to accept the pre-conditions' types as inputs, and then encodes their boolean function checks within the constructor to verify that all of the inputs are valid. 
Tus the constructor either returns an Optional value of the successfully created datatype, or an Error specifying which of the pre-conditions failed to validate (and the corresponding input value that made it fail). In this way, if the datatype succeeds in being constructed, the developer has the guarantee (based off of the interface definition spec) that the inputs are all valid.

However there is one key thing missing in our interface definition to allow for this kind of scheme to work. We have defined all of the required inputs possible for the given smart contract, however we have not considered the fact that only a sub-set of these inputs may be required for any given _action_ which a participant in a smart contract can perform.

As such we have to define said actions, which in essence are simply a grouping of pre-conditions put together. (The astute among you will realize this also clearly maps onto structs/datatypes.) 

A dApp will often have more than one action, but we will keep things simple and explore what defining a single one would look like.

IDL pseudocode:
```
let lockAmount = Int.range(1,10);
let lockPassword = String.maxSize(35);
let betterLockPassword = String.minSize(lockPassword.length + 5);

action lockFunds(lockAmount, lockPassword, betterLockPassword);
```

What this means is that an action called `lockFunds` is defined as a valid action for the smart contract which takes 3 input values. Each input must be of the correct type (ie. Int, String), and must also pass the defined pre-condition in order for it to be valid.

This IDL can then be transformed into a struct/datatype via meta-programming as shown in the pseudo-code below:

```
struct LockFundsAction = {
    lockAmount: Int,
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
                betterLockPassword: betterLockPassword,
            }
        }
    }
}
```

Thus we now have an input validator created via meta-programming based off of our IDL.

_Note: From the implementor's perspective of this more advanced IDL, the pre-conditions must now pass on their core type requirement as well as the boolean function. In languages that have first-class types, you could just make the results from the pre-condition generation functions be a tuple which is made of (type, boolean function) for example. However as this isn't the case for most languages, there may need to be a bit more work done behind-the-scenes in your given implementation language to have type inference from the boolean function._


Automatic GUI Generation
-------------

Now with an action struct created, it is trivial to automatically generate GUI elements.

A user will be provided with a graphical list of actions based off of the actions defined in the interface definition. When a user selects an action, they are selecting the corresponding struct/datatype which we will use to validate their input.

To create the actual GUI, the parser simply needs to read the types of the fields of the struct, and with that build graphical user interface elements which allow the user to submit values of the corresponding type (Int, String, ...). Furthermore the field name + the pre-conditions represented in textual format can be placed above each GUI element, thereby explaining to the end-user what is required.

Once the values are submitted by the user, they can be used as input to the action struct's constructor (`new()`). If the user submitted valid input, then everything is good, else the error will be returned and can be displayed in textual format back to the user so they can fix their mistake.

But we can still do a bit better. By expanding on our IDL we can allow developers to specify ideal GUI elements for an input. This would improve the auto-generation process if we wish to add such enhanced support.

Example pseudo-code:
```
# lockAmount.GUI: Slider
let lockAmount = Int.range(1,10);

# lockPassword.GUI: InputBox
let lockPassword = String.maxSize(35);
```
With that all said and done we are now able to automatically generate GUI and verify that the inputs provided are valid.

However there is one final piece missing to make this work truly flawlessly. We have not yet specified how the inputs are to be used in performing the actual action (building a transaction with the given inputs).

Performing An Action
-------------

To perform an action we will have to encode how a transaction is to be created using the provided inputs. 
This often is easiest using a json-like format and specifying all the relevant parts of a transaction.

Here is a pseudo-example of how this could work in our IDL:
```
# lockAmount.GUI: Slider
let lockAmount = Int.range(1,10);
# lockPassword.GUI: InputBox
let lockPassword = String.maxSize(35);

action lockFunds(lockAmount, lockPassword) {
    return ({
            script = dApp.smartContract("lockingSmartContact.sc"),
            register.R4 = lockPassword,
            amount = lockAmount
        })
}
```
_(As can be seen, I also included the concept of using the smart contract which is included as a separate file named `lockingSmartContract.sc` within the same `.dApp` package that this interface definition a part of.)_

To put it into words, our `lockFunds` action takes in 2 inputs, `lockAmount` and `lockPassword`, and when performed it produces a transaction with values as specified above.

This then can be translated into a new method for building the transaction for our `LockFundsAction` struct:

```
impl LockFundsAction {
    fn buildTransaction (self) -> Transaction {
        // buildAndSignTransactionWithData is just a placeholder function for an equivalent that works with one's given ecosystem/toolchain
        let tx = buildAndSignTransactionWithData({
                    script = dApp.smartContract("lockingSmartContact.sc"),
                    register.R4 = self.lockPassword,
                    amount = self.lockAmount});
        return(tx);
    }
}
```

At this point in time, our action struct now can be created (with all of the input validity checking that goes with it), and then performed by building the corresponding transaction which gets submitted to the network.

Thus we have created a generalized interface for interacting with the dApp that is easily usable by both automatically generate GUIs as well as libraries/frameworks which can plug right in and provide an entire development environment.

It is therefore possible (with an IDL like this) to one day have wallets allow users to import a `.dApp` file, automatically generate the GUI, guarantee input validity, and then build the transaction which is finally passed on to the wallet to be submitted to the network seamlessly. A much more idealized future compared to what we have today in the dApp world if I do say so myself.

Packaging Other Useful Files
----------

One last point to dive into before we conclude, is that we can also add other files to the package to enrich the dApp even further. Some basic possibilities:
- Configuration file 
- Icons
- Formal specification/proofs (which anyone can validate) 
- Package signature by the author
- ...

In the configuration file we can include descriptive elements such as the dApp title, version, description, copyright, author details, or any technical details that are relevant (ex. the language of the smart contract or formal spec, specifying runtime/parser, ...).

With each useful file/piece of information, we enrich the dApp with more context and possibilities.

Going Forward
----------

This document is primarily a high-level overview of the concept of a Universal dApp Interface & Package Standard, rather than a specific definition of one. Chances are that each given platform/project will have to create a custom standard for themselves based on the requirements/tooling in the ecosystem at hand. Nonetheless, it is my general contention that on the high-level such a standard is one of the better ways forward for dApp development at large.

A few of us in the [Ergo](https://ergoplatform.org) community have already began to ruminate on how a Universal dApp Interface & Package Standard could be integrated within [AppKit](https://github.com/aslesarenko/ergo-appkit) to achieve `.dApp` support in the primary cross-language polyglot dApp library for the Ergo ecosystem, and also how it could be integrated with [Stainless](https://github.com/epfl-lara/stainless) to improve the formal verification process of smart contracts. In short, the possibility of implementing such a dApp standard may not be that far off in the future, which is quite exciting from my perspective.

I have no doubt that there are major improvements possible on my design of the IDL (this idea is very nascent), so please feel free to send me a message if you have any thoughts/questions/ideas/criticisms on this.
