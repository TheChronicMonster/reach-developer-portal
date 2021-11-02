---
menuItem: mi-docs
---

# Coin Flipper

# Introduction

This tutorial walks you through how to build a command-line version of a Decentralized Application (DApp) that simulates a Coin Flip and allows two players to bet if the result will be heads or tails. 

This DApplication initializes two participants, Alice and Bob. Alice chooses a face of the coin, Bob confirms the choice and sets the wager amount. Alice confirms the wager amount and then the coin is flipped, the outcome is seen by both parties and then funds are exchanged.

## Learning Objectives

By the end of this tutorial you will:

1. Understand how to initiate participants.
2. Communicate data between Reach and the front-end.
3. Build a Reach command-line DApp.

## Overview



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

3. Use VSCode or your favorite text editor to open the contents of the `current` directory. _This tutorial was built using VSCode, but any modern browser should offer the Reach extension package._

## Review index.mjs starter

``` js
load: https://github.com/TheChronicMonster/coinflipper/blob/main/starter/index.mjs
```

Let's take a look through this starter code:

* Line 1: Import the Reach JS Standard Library loader. (A common first step).
* Line 2: Import the JS backend compiled from index.rsh (A common second step).
* Line 3: loads the Standard Library dynamically based on the [REACH_CONNECTOR_MODE](https://docs.reach.sh/ref-usage.html#%28env._.R.E.A.C.H_.C.O.N.N.E.C.T.O.R_.M.O.D.E%29) environment variable.
* Line 5: Defines an asynchronous function to await the fulfillment of promises.
* Line 6: Defines a quantity of network tokens as the starting balance for Alice and Bob. The `parseCurrency` method transforms units from standard to atomic.
* Line 7 and 8: Creates test accounts with the startingBalance endowments for Alice's and Bob's accounts.
* Line 10: Creates a reference to the contract.
* Line 11: Bob references the contract and utilizes a method to get information from Alice.
* Line 13: Creates a promise that waits for the backend to complete.
* Line 14 - 16: Initalizes a backend for Alice.
* Line 17 - 19: Initalizes a backend for Bob.
* Line 21: Calls the asynchronous function that was defined on line 5.

## Review index.rsh starter

``` js
load: https://github.com/TheChronicMonster/coinflipper/blob/main/starter/index.rsh
```

Let's look at the Reach code:

* Line 1: Instructs the compiler to use Reach version 0.1.0. This is the first line in every Reach program.
* Line 3: This inializes the standard Reach application. It is the main export of the program and is what the complier looks at when compiling. 
* Line 4 - 9: Specifies two participants to represent players in the DApp. Alice and Bob are common participant names in cybersecurity exercises.
* Line 10: Finalizes the particpant and any other options and marks the deployment of the Reach program.

# Run the DApp

You now have the basic syntax required to deploy a Reach DApp. Be sure Docker is running, and then in the terminal, run this initial code using:

```$ ./reach run```

Reach will run a Docker container and compile the app. However, at this point in time, the application doesn't do anything so you'll only see diagnostic messages.

# Implement Logic

In this next phase, we'll create the basic logic for the coin flipper and create command-line outputs. This will make your application more exciting!

## Developing with a Reach Mindset

When creating a Reach application, it's important to think about the participants (users) and how they will interact with the DApp. If you're coming from a background in software engineering or full stack development then you might be more accustomed to thinking in terms of how the application is going to work and interact with APIs. 

While that is a consideration in Reach, your primary mindset should be centered on how the user will use the product. Think user first to aid you in the development of your Reach applications.

## Thinking through user interactions

1. For this dapp, there will be two participants betting on a coin flip.

2. Alice (Participant A) will deploy the dapp and chose the face of the coin to bet on. 

3. Bob (Participant B) will connect to the contract, confirm the face choice, and set the wager amount.

4. Alice will confirm the wager amount and both participants will see the face and wager confirmations.

5. A coin flip will be simulated using a mathematical calculation.

6. The outcome will be revealed and payments made.

## Creating constants

Now that we have a plan for how the participants will interact with the dapplication, we can create the constants.

Insert this snippet in _index.mjs_ below the contract (ctc) constants.

``` js
const FACE = ['Heads', 'Tails'];
const COIN = ['Heads', 'Tails'];
const OUTCOME = ['Bob wins', 'Alice wins'];
const Player = (Who) => ({
    // Objects will go here
})
```

* Line 1: Creates the face constant which will allow Alice to choose Heads or Tails.
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
    const Alice = Participant('Alice', {
        // interface here
        ...Player,
    });
    const Bob = Participant('Bob', {
        // interface here
        ...Player,
    });
    deploy();
```

Line 5: Creates the participant interact interface
Line 6 and 7: Methods that take no parameters and returns a string.
Line 8: A method that accepts an integer.

Line 13 - 21: Alice and Bob will be able to interact with the front-end because of this arrow function. `...Player` adds all `Player` properties to the participant interface.

Let's jump over to _index.mjs_ to add two lines in the interact object.

Find the Promise `await Promise.all([...])` so that we can instantiate the implementations of Alice and Bob in each of their backends. The following addition will allow the frontend participants to interact with the backend.

``` js
await Promise.all([
    backend.Alice(ctcAlice, {
        ...Player('Alice'),
    }),
    backend.Bob(ctcBob, {
        ...Player('Bob'),
    }),
]);
```

Let's take this moment to discuss the "Participant Interact Interface." As the name implies, this is the interface that allows participants to interact with one another. The PII is an object that dictates the functions and values that the frontend will provide to the backend in order to interact with each particiapnt. 

The PII is a core component of any Reach program. PIIs are made up of participants and participant classes. Each participant name must be unique.

It is common in cryptology to name participants in order of Alice, Bob, Carol, Dave, etc. An eavesdropper is named Eve, malicious attackers are named Mallory, Intruders are known as Trudy, Whistle-blowers are Wendy, and provers and verifiers are Peggy and Victor, but cryptographers who are also lovers of popular game shows may use Pat and Vanna.

Now that we've defined our methods for the participant interact inferface in the reach file, let's add the objects to our Player class.

Find the following code block in _index.mjs_ 

``` js
const Player = (Who) => ({
});
```

We will add a JavaScript object for each function in the PII. Remember, our DApp will allow Alice to choose a face of the coin, compute a coin toss to randomly determine heads or tails and then print the outcome of the result. 

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
Line 3: is a temporary statement that helps Alice randomly choose Heads or Tails.
Line 4: prints Alice's choice.
Line 5: is a temporary message that shows Alice's choice as a "0" for heads or "1" for tails
Line 6 - 10: An if/else expression that determines the value of `faceName`. 
Line 11: Returns the value in `faceName`.

Now well create the `tossCoin` object. The purpose of `tossCoin` will be to simulate a coin toss and store and return the value of the result.

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

Our next area of focus will be in _index.rsh_ where we will tell Reach what actions Alice and Bob are able to perform and what information can be shared. In this section we will be introduced to the `declassify`, `commit`, and `publish` methods, as well as, the `interact` property. 

These methods and property are cruical to participant interaction.

--------
**Terminology**
[declassify](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28declassify%29%29%29): declassifies a given argument

[commit](https://docs.reach.sh/ref-programs-consensus.html#%28reach._%28%28commit%29%29%29): ends the consensus step and allows for additional local steps

[publish](https://docs.reach.sh/ref-programs-step.html#%28reach._%28%28publish%29%29%29): Publishes new data

[interact](https://docs.reach.sh/ref-programs-local.html#%28reach._%28%28interact%29%29%29): Evaluates the results of an interaction with the frontend. May only occur in a local step
--------

Return to _index.rsh_ and enter the following snippet after the `deploy()` statement. This snippet will be inside the Reach Application. (before the closing syntax)

``` js
Alice.only(() => {
    // Methods that only Alice can perform
});

Bob.only(() => {
    // Methods that only Bob can perform
});
```

Lines 1 - 3: This is a block of code that only Alice can perform.
Lines 5 - 7: This is a block of code that only Bob can perform.


``` js
Alice.only(() => {
    const faceAlice = declassify(interact.chooseFace());
});
Alice.publish(faceAlice);
commit();

Bob.only(() => {
    const tossResult = declassify(interact.tossCoin());
});
Bob.publish(tossResult);

const outcome = (faceAlice == tossResult) ? 1 : 0; // 0: Bob wins; 1: Alice wins
commit();

each([Alice, Bob], () => {
    interact.seeOutcome(outcome);
});