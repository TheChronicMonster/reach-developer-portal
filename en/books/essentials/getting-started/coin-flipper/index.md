---
menuItem: mi-docs
---

# Coin Flipper

<<<<<<< HEAD
# Introduction

This tutorial walks you through how to build a command-line version of a decentralized application (DApp) that simulates a Coin Flip and allows two players to bet if the result will be heads or tails. 

This DApplication initializes two participants, Caller and Flipper. Caller chooses a face of the coin. Flipper confirms the choice and sets the wager amount. Caller confirms the wager amount and then the coin is flipped. The outcome is seen by both parties and then funds are exchanged.

## Learning Objectives

By the end of this tutorial you will:

1. Understand how to initialize participants.
2. Communicate data between the backend and the frontend.
3. Build a Reach command-line DApp.

## Overview

[Coin Flipper Role Card](https://www.canva.com/design/DAEvbyYCtSE/HKEJ79IKOrMOIH51dyXQdA/view?utm_content=DAEvbyYCtSE&utm_campaign=designshare&utm_medium=link&utm_source=sharebutton)

# Getting Started

## Clone the repository

This section shows you how to clone the tutorial repository:

1. Clone the [coin-flipper](https://github.com/TheChronicMonster/coinflipper) repository:

``` nonum
$ cd ~/reach
$ git clone https://github.com/TheChronicMonster/coinflipper.git
```

2. cd into the `coinflipper` directory and copy the starter files to the `current` directory.

``` nonum
$ cd coinflipper
$ cp starter/index.rsh current/index.rsh
$ cp starter/index.mjs current/index.mjs
```

3. Open the contents of directory, "`current`".

## Review index.mjs starter

``` js
load: https://github.com/TheChronicMonster/coinflipper/blob/main/starter/index.mjs
```

Let's take a look through this starter code:

* Line 1: Import the Reach JS Standard Library loader. (A common first step).
* Line 2: Import the JS backend compiled from index.rsh (A common second step).
* Line 3: loads the Standard Library dynamically based on the [REACH_CONNECTOR_MODE](https://docs.reach.sh/ref-usage.html#%28env._.R.E.A.C.H_.C.O.N.N.E.C.T.O.R_.M.O.D.E%29) environment variable.
* Line 5: Defines an asynchronous function to await the fulfillment of promises.
* Line 6: Defines a quantity of network tokens as the starting balance for Caller and Flipper. The `parseCurrency` method transforms units from standard to atomic.
* Line 7 and 8: Creates test accounts with the startingBalance endowments for Caller's and Flipper's accounts.
* Line 10: Creates a reference to the contract.
* Line 11: Flipper references the contract and utilizes a method to get information from Caller.
* Line 13: Creates a promise that waits for the backend to complete.
* Line 14 - 16: Initalizes a backend for Caller.
* Line 17 - 19: Initalizes a backend for Flipper.
* Line 21: Calls the asynchronous function that was defined on line 5.

## Review index.rsh starter

``` js
load: https://github.com/TheChronicMonster/coinflipper/blob/main/starter/index.rsh
```

Let's look at the Reach code:

* Line 1: Instructs the compiler to use Reach version 0.1.0. This is the first line in every Reach program.
* Line 3: This inializes the standard Reach application. It is the main export of the program and is what the complier looks at when compiling. 
* Line 4 - 9: Specifies two participants to represent players in the DApp. Caller and Flipper are common participant names in cybersecurity exercises.
* Line 10: Finalizes the particpant and any other options and marks the deployment of the Reach program.

# Run the DApp

You now have the basic syntax required to deploy a Reach DApp. Be sure Docker is running, and then in the terminal, run this initial code using:

```$ ./reach run```

Reach will run a Docker container and compile the app. However, at this point in time, the application doesn't do anything so you'll only see diagnostic messages.

# Implement Logic

In this next phase, we'll create the basic logic for the coin flipper and create command-line outputs. This will make your application more exciting!

## Developing with a Reach Mindset

When creating a Reach application, it's important to think about the participants (users) and how they will interact with the DApp. If you're coming from a background in software engineering or full stack development then you might be more accustomed to thinking in terms of how the application is going to work and interact with APIs.

## Thinking through user interactions

1. For this DApp, there will be two participants betting on a coin flip.

2. Caller (Participant A) will deploy the DApp and choose the face of the coin to bet on. 

3. Flipper (Participant B) will connect to the contract, confirm the face choice, and set the wager amount.

4. Caller will confirm the wager amount and both participants will see the face and wager confirmations.

5. The frontend will simulate a coin flip using a mathematical calculation.

6. The outcome will be revealed and payments made.

## Creating constants

Now that we have a plan for how the participants will interact with the dapplication, we can create the constants.

Insert this snippet in _index.mjs_ below the contract (ctc) constants.

``` js
const FACE = ['Heads', 'Tails'];
const COIN = ['Heads', 'Tails'];
const OUTCOME = ['Flipper wins', 'Caller wins'];
const Player = (Who) => ({
    // Objects will go here
})
```

* Line 1: Creates the face constant which will allow Caller to choose Heads or Tails.
* Line 2: Creates the coin constant which will be used to simulate the coin flip.
* Line 3: Creates an outcome constant to reveal who won the coin toss.
* Line 4: The Player constant which will hold Reach objects (currently empty)

Next, we'll edit the reach file, _index.rsh_, so reach can interact with the front-end.

``` js
'reach 0.1'

// Create participant interact interface

const Player = {
    chooseFace: Fun([], Bytes(6)),
    tossCoin: Fun([], Bytes(6)),
    seeOutcome: Fun([UInt], Null),
};

// instantiate two players

export const main = Reach.App(() => {
    const Caller = Participant('Caller', {
        // interface here
        ...Player,
    });
    const Flipper = Participant('Flipper', {
        // interface here
        ...Player,
    });
    deploy();
```

Line 5: Creates the participant interact interface
Line 6 and 7: Methods that take no parameters and return strings.
Line 8: A method that accepts an integer.

Line 13 - 21: Caller and Flipper will be able to interact with the front-end because of this arrow function. `...Player` adds all `Player` properties to the participant interface.

Let's jump over to _index.mjs_ to add two lines in the interact object.

Find the Promise `await Promise.all([...])` so that we can instantiate the implementations of Caller and Flipper in each of their backends. The following addition will allow the frontend participants to interact with the backend.

``` js
await Promise.all([
    backend.Caller(ctcCaller, {
        ...Player('Caller'),
    }),
    backend.Flipper(ctcFlipper, {
        ...Player('Flipper'),
    }),
]);
```

The Participant Interact Interface is the interface that allows participants to interact with one another. The interact object is an interface that dictates the functions and values that the frontend will provide to the backend in order to interact with each participant. 

This is a core component of any Reach program. The Participant Interact Interface is made up of participants and participant classes. Each participant name must be unique.

It is common in cryptology to name participants in order of Caller, Flipper, Carol, Dave, etc. An eavesdropper is named Eve, malicious attackers are named Mallory, Intruders are known as Trudy, Whistle-blowers are Wendy, and provers and verifiers are Peggy and Victor, but cryptographers who are also lovers of popular game shows may use Pat and Vanna. _[source](http://www.nancy.cc/2016/08/23/cryptography-names-alice-bob-eve/)_

Now that we've defined our methods for the inferface in the reach file, let's add the objects to our Player class.

Find the following code block in _index.mjs_ 

``` js
const Player = (Who) => ({
});
```

We will add a JavaScript object for each function in the interact object. Remember, our DApp will allow Caller to choose a face of the coin, compute a coin toss to randomly determine heads or tails and then print the outcome of the result. 

Let's take on each object, one at a time.

``` js
chooseFace: () => {
    let faceName = "";
    const face = Math.floor(Math.random() * 2);
    console.log(`${Who} chose ${FACE[face]}`);
    console.log(`The constant face is ${face}`);
    if (face == 0) {
        faceName = "Heads";
    } else {
        faceName = "Tails";
    }
    return faceName;
},
```

Line 1: Creates the chooseFace object.
Line 2: Creates an empty string in the `faceName` variable. 
Line 3: is a temporary statement that helps Caller randomly choose Heads or Tails.
Line 4: prints Caller's choice.
Line 5: is a temporary message that shows Caller's choice as a "0" for heads or "1" for tails
Line 6 - 10: An if/else expression that determines the value of `faceName`. 
Line 11: Returns the value in `faceName`.

Now we'll create the `tossCoin` object. The purpose of `tossCoin` will be to simulate a coin toss and store and return the value of the result.

We'll begin this object on the next line after `chooseFace` closes.

``` js
tossCoin: () => {
    let coinName = "";
    const coin = (Math.floor(Math.random() * 2) == 0);
    if (coin) {
        coinName = "Heads";
    } else {
        coinName = "Tails";
    }
    console.log(`The coin flip reveals ${coinName}`);
    console.log(`The coin number is ${coin}`);
    return coinName;
},
```

Line 1: Creates the tossCoin object.
Line 2: Creates an empty string in the `coinName` value.
Line 3: is a simple calculation that checks if a result randomly equals 0.
Line 4 - 8: is an if/else statement that assigns the value of "Heads" or "Tails" based on the value of `coin`.
Line 9: Prints the outcome of the flip as "Heads" or "Tails"
Line 10: Prints the outcome of the flip as "true" or "false"

Finally, let's create the `seeOutcome` object, on the next line after the `tossCoin` object closes.

``` js
seeOutcome: (outcome) => {
    console.log(`${Who} saw outcome ${OUTCOME[outcome]}`);
},
```

Line 1: Instantiates the `seeOutcome` object.
Line 2: Prints a message that the players saw the outcome of the coin toss.
Line 3: Closes the object.

At this point in the program, we have created the basic building blocks of the business logic. The program will run, but information will not be published to the user. 

Our next area of focus will be in _index.rsh_ where we will tell Reach what actions Caller and Flipper are able to perform and what information can be shared. In this section we will be introduced to the `declassify`, `commit`, and `publish` methods, as well as, the `interact` property. 

These methods and property are crucial to participant interaction.

--------
**Terminology**
[declassify](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28declassify%29%29%29): declassifies a given argument

[commit](https://docs.reach.sh/ref-programs-consensus.html#%28reach._%28%28commit%29%29%29): ends the consensus step and allows for additional local steps

[publish](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28publish%29%29%29): Publishes new data

[interact](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28interact%29%29%29): Evaluates the results of an interaction with the frontend. May only occur in a local step
--------

Return to _index.rsh_ and enter the following snippet after the `deploy()` statement. This snippet will be inside the Reach Application. (before the closing syntax)

``` js
Caller.only(() => {
    // Methods that only Caller can perform
});

Flipper.only(() => {
    // Methods that only Flipper can perform
});
```

Lines 1 - 3: This is a block of code that only Caller can perform.
Lines 5 - 7: This is a block of code that only Flipper can perform.

Now we will add variables to Caller and Flipper and define an outcome.

``` js
Caller.only(() => {
    const faceCaller = declassify(interact.chooseFace());
});
Caller.publish(faceCaller);
commit();

Flipper.only(() => {
    const tossResult = declassify(interact.tossCoin());
});
Flipper.publish(tossResult);

const outcome = (faceCaller == tossResult) ? 1 : 0; // 0: Flipper wins; 1: Caller wins
commit();

each([Caller, Flipper], () => {
    interact.seeOutcome(outcome);
});
```

Line 2: declassifies Caller's face and saves the value in `faceCaller`
Line 4: Publishes `faceCaller` in the consensus network
Line 5: Commits Caller's local step to the consensus network
Line 8: Flipper declassifies the result of the coin toss and saves the value in `tossResult`
Line 10: Flipper publishes the value in `tossResult`
Line 12: Creates a conditional statement that results in a win for Caller when true and a win for Flipper when false
Line 15: A local step that each of the participants can perform

You can now run the program. Here are results from a few of my playthroughs:

``` bash
$ reach run
Caller chose Tails
The constant face is 1
The coin flip reveals Heads
The coin number is true
Flipper saw outcome Flipper wins
Caller saw outcome Flipper wins
```

``` bash
$ reach run
Caller chose Heads
The constant face is 0
The coin flip reveals Tails
The coin number is false
Flipper saw outcome Flipper wins
Caller saw outcome Flipper wins
```

``` bash
$ reach run
Caller chose Heads
The constant face is 0
The coin flip reveals Heads
The coin number is true
Flipper saw outcome Caller wins
Caller saw outcome Caller wins
```

Two out of three for Flipper is not bad! 

Caller and Flipper don't need to know one another, and it's not required that they've actually met in order to trust that they both see the same outcome. This is the purpose of a consensus network: the ability to allow globally distributed and untrusted parties to reach an agreement throughout every state in a program. 

Flipping a coin is great, but we can make the game more interesting by YOLOing all our crypto on a coin flip.

# Adding Wagers

We'll add new functionality to the coin flipper dApp such that participants will be able to make a wager on the coin flip. This will require that we aded a new integer called `wager` for Caller and an `acceptWager` method that can use `wager` as a paramter.

We will add the value and method to the participant interact interface in the backend. 

``` js
'reach 0.1';

const Player = {
    chooseFace: Fun([], Bytes(6)),
    tossCoin: Fun([], Bytes(6)),
    seeOutcome: Fun([UInt], Null),
};

export const main = Reach.App(() => {
    const Caller = Participant('Caller', {
        ...Player,
        wager: UInt,
    });
    const Flipper = Participant('Flipper', {
        ...Player,
        acceptWager: Fun([UInt], Null),
    });
    deploy();

    Caller.only(() => {
        // ...
```

Line 12: Adds the `wager` integer value
Line 16: Flipper is given an `acceptWager` method which can accept `wager` as a parameter and returns Null.

Now we'll update Caller so that she'll be able to declassify her wager amount, publish her wager to the consensus network and pay.

``` js
Caller.only(() => {
    const wager = declassify(interact.wager);
    const faceCaller = declassify(interact.chooseFace());
});
Caller.publish(wager, faceCaller)
  .pay(wager);
commit();
```

Line 2: declassifies the wager value so that it may be communicated to the consensus network
Line 5: Adds wager to the publish method so that Caller will share the value with Flipper
Line 6: pay is a method that transfers the wager amount when Caller publishes her wager and coin face. 

Now we'll update Flipper:

``` js
Flipper.only(() => {
    interact.acceptWager(wager);
    const tossResult = declassify(interact.tossCoin());
});
Flipper.publish(tossResult)
  .pay(wager);
// ...
```

Line 2: Flipper is able to accept Caller's wager. This version of the dApp has a vulnerability, in that if Flipper doesn't like Caller's wager he can simply stop interacting with the program.
Line 6: Flipper matches Caller's wager

Before we write the computation, let's return to the frontend to give Caller and Flipper the ability to see their balances before and after the wagers.

``` js
(async () => {
const startingbalance = stdlib.parseCurrency(10);
const accCaller = await stdlib.newTestAccount(startingBalance);
const accFlipper = await stdlib.newTestAccount(startingBalance);

const fmt = (x) => stdlib.formatCurrency(x, 4);
const getBalance = async (who) => fmt(await stdlib.balanceOf(who));
const beforeCaller = await getBalance(accCaller);
const beforeFlipper = await getBalance(accFlipper);

const ctcCaller = accCaller.contract(backend);
const ctcFlipper = accFlipper.contract(backend, ctcCaller.getInfo());
// ...
```
Line 6: Formats to show standard currency with 4 decimal places
Line 7: Gets the balance for each participant and show in a standard format with up to 4 decimal places
Line 8 & 9: Retrieves the balance before Caller and Flipper play

------
acc is shorthand for account
ctc is shorthand for contract
------

Now find where Caller's interface object begins and add a wager object.

``` js
await Promise.all([
    backend.Caller(ctcCaller, {
        ...Player('Caller'),
        wager: stdlib.parseCurrency(5),
    }),
// ...
```

Line 3: Adds all of the common Player methods into Caller's interface
Line 4: Defines every wager as 5 atomic network token units. Atomic units are always integers.

Flipper's interface object will be updated to show the wager amount and automatically accept it. In a more robust version we will want Flipper to have the ability to decline the wager. For this version, however, Flipper will simply accept.

``` js
backend.Flipper(ctcFlipper, {
    ...Player('Flipper'),
    acceptWager: (amt) => {
        console.log(`Flipper accept the wager of ${fmt(amt)}.`);
    },
// ...
```

Line 2: Adds the common Player interface into Flipper's interface
Line 3 - 5: Defines the acceptWager function, which returns a message to the console.

Next, we'll store the new balance amounts in constant variables and then print statements to the console showing the before and after token amounts for Caller and Flipper. 

``` js
const afterCaller = await getBalance(accCaller);
const afterFlipper = await getBalance(accFlipper);

console.log(`Caller went from ${beforeCaller} to ${afterCaller}.`);
console.log(`Flipper went from ${beforeFlipper} to ${afterFlipper}.`);
// ...
```

Lines 1 & 2: Stores the value of the balance for Caller and Flipper
Lines 3 & 4: Prints the before and after balances for Flipper and Caller

This completes our updates for our front end. The final step for this portion is to create the payment transfer logic. Return to _index.rsh_ and add this code after Flipper's interface object and before the `each` statement.

``` js
const outcome = (faceCaller == tossresult) ? 1 : 0;
const            [forCaller, forFlipper] =
  outcome == 1 ? [       2,      0] :
                 [       0,      2];
transfer(forCaller * wager).to(Caller);
transfer(forFlipper * wager).to(Flipper);
commit();
// ...
```

Lines 2 - 4: The outcome determines who receives a payout. If the outcome is equal to 1 then Caller receives the payout, whereas if the outcome is 0 then Flipper receives the payout. 
Lines 5 - 6: Transfer money according to the outcomes in lines 2 - 4. 
Line 7: The commit method commits the state of the applicaiton and publishes the results of the outcome to the participants. 

You can now run the program with `$ reach run`. Your results may vary as the program acts randomly every time. 

Here are three of my results.

```bash
$ reach run
Caller chose Tails
The constant face is 1
Flipper accepts the wager of 5.
The coin flip reveals Tails
The coin number is false
Flipper saw outcome Caller wins
Caller saw outcome Caller wins
Caller went from 10 to 14.996.
Flipper went from 10 to 4.996.
```

``` bash
$ reach run
Caller chose Tails
The constant face is 1
Flipper accepts the wager of 5.
The coin flip reveals Tails
The coin number is false
Flipper saw outcome Caller wins
Caller saw outcome Caller wins
Caller went from 10 to 14.996.
Flipper went from 10 to 4.996.
```

``` bash
$ reach run
Caller chose Tails
The constant face is 1
Flipper accepts the wager of 5.
The coin flip reveals Heads
The coin number is true
Flipper saw outcome Flipper wins
Caller saw outcome Flipper wins
Caller went from 10 to 4.996.
Flipper went from 10 to 14.996.
```

Flipper didn't do as well once we began to wager our coin flips. Some people get nervous once crypto is involved. 
=======
<span style="color:red;">Under Construction</span>
>>>>>>> ed083b737346f3469102bcc8cfc18e5811693585
