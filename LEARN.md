# Building a Russian Roulette dapp on Ethereum
Hi! Glad to see you here!  
In this quest, you will create a cool game on Ethereum, it is Ethereum’s version of Russian Roulette. You may know it from movies or Russian literature. It is a game in which two players take turns in, well, shooting themselves. Each player has a gun with only one bullet in its chamber, then he/she rolls and shoots and then passes the turn. Obviously, the one who is still alive wins. 

So, we are going to write a smart contract using Solidity and a basic UI to interact with it. The UI is built using react.js, we assume a basic knowledge of how react works. 

The code for this quest is available on [https://github.com/TheLedgerOfJoudi/russian-roulette](https://github.com/TheLedgerOfJoudi/russian-roulette) (solidity code is in the main branch and react.js snippets in master).  Let’s get our hands on the keyboard!
## Writing the smart contract - setup and state variables
First, we need to define the compiler version(s) that our contract will use. That is done in the first line of code right after defining the license, in this quest we use compiler versions between 0.4.22 (included) and 0.8.0 (excluded). Choosing which compiler version to use is always a philosophical question, I prefer to choose a range of versions that is not quite old. Also, if you don’t specify the SPDX license identifier there is a good chance that your compiler will yell at you. We use an MIT license so anyone can take the code and use it. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/48afdf5a-fccc-4302-bf5b-b3afa42496ff.jpg)

Now that we have our version license and version set, let’s write the contract. We need a data structure to hold players’ data, a fixed-size array of size two seems like a good option (array players in code). The `payable` keyword is to indicate that we want these addresses to receive money. 

We also need a utility variable to point at the current index in the array (`uint8 index` in code). We chose uint8 (8 bit integer) because this variable can only take 0 or 1, so 8 bits are enough. The more optimized our contracts are in storage, the cheaper it is to deploy them. Then there is another variable that indicates whose turn it is `uint turn`. We will see later why we kept this variable (and others) as type uint. The variable named `killed` is there to randomly determine which of the two holes has the bullet in it (each gun chamber has two holes in our game, just to keep it short). Finally, the bool finished is used to indicate whether the game is over or not.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/f628ad5f-253b-4a27-b325-0f9a1d5c3414.jpg)

Now, take a breath and get ready to write the game’s functions!
## Creating a random number
In our constructor, we have to assign two things, which player is starting to shoot and which hole has the bullet in it. Those two variables have to be initialized randomly each time. Creating random numbers is a big issue in Solidity, it is really hard to design a function that returns a “good” random value. But for the sake of simplicity, we will use a commonly used way to generate pseudo randomness in Solidity. That is what the weird-looking lines in the constructor are doing. It is taking those bunch of parameters, encoding them to bytes, applying the Keccak256 hashing algorithm, casting the resulting hash to uint, and then taking the result mod 2. If you change turn to be a uint8 your code will fail because of type conversions, but let’s not get lost in ugly details and remember that the whole purpose is to get a random value, depending on what the current block is.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/04088150-e1f0-4efe-bc43-d767a813c8a0.jpg)

Cool, now we know who is going to start and where the bullet is. It is time to register players to start the game!

Subquest : Using modifiers to keep the code clean

In function `register()`, we first make sure that there are only two players in the game using the index variable. Then require that the player pays 0.5 ether to enter the game. Of course, the player will pay that in hope that he/she will win and get the loser’s 0.5 ether. Of course, the function has to be payable for that. Then comes some simple logic, just acknowledge that `msg.sender` - the address of the user who is calling this function - is now registered and increase the index to accommodate that next player if any. 

  
Then we add a simple check if two players are registered, then the game is on again.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/ff95df67-9461-4c4b-90dd-89f09049858e.jpg)

This next chunk of code contains some useful utilities. An event that our UI can subscribe to, will come in handy later. Two function modifiers to help the contract run the game. one to check if it is a player’s turn indeed and one to check if the game is still on. You can attach function modifiers to functions - that way, the function call is executed via this middleware.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/9fd79492-56c1-4b64-b0eb-222f36cc3627.jpg)
## Broadcasting an event
Now let’s move on to the main function, shoot()!

It is also really simple, this function checks if the game is not finished and that it is the turn of the one calling the function, generates a random shot, and checks if it matches the killed variable (the bullet). If that happens, then it emits the event with a loser as a parameter, sends the funds to the winner, and resets state variables. By emitting an event, any UI listening to the event can make required changes.

If not killed, simply pass the turn. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/f8b93e62-29de-49ce-92be-68aeacb13fd0.jpg)

Then some useful functions, one to get the address of the current player and one to check game status.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/ac6869a5-23f8-497a-b04c-09e07df68cef.jpg) 

Alright, now our contract is complete and it is time to create the interface!
## Writing the UI - setting up
Before starting, remember that you can find the whole code on [https://github.com/TheLedgerOfJoudi/russian-roulette/tree/master](https://github.com/TheLedgerOfJoudi/russian-roulette/tree/master) . Also, this is what our UI looks like : 

[https://theledgerofjoudi.github.io/russian-roulette/](https://theledgerofjoudi.github.io/russian-roulette/)

 Ok, so after you create your react app using 

```

npx create-react-app name_of_your_app

cd name_of_your_app

```

Download `web3` which is one of the libraries used to interact with the blockchain from the browser.

```

npm install web3 --save

```

  
Make sure to have this in your App.css:

```

.App{

Text-align : center;

}

```

 Again, this is not a front-end quest, but our UIs already suck and it is always a good idea to make them suck less. Inside your src folder, create a folder called components, this is where we will keep our interface’s components. Inside the component, create four .js files as in the image below:

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/212f58c0-a453-4b24-879b-acec39656595.jpg)

Now, go back to the src folder and create a config.js file.

[https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/config.js](https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/config.js) 

 In this file create two constants like this. Compile the Solidity file on Remix and tap on “ABI” to copy the ABI and paste it in the `config.js`. Then go on to deploy, deploy your contract to Ropsten using injected web3 & copy the contract address.

```

export const ADDRESS = “put the contract’s address here right after you deploy it”  
 export const ABI = \[//put here the contract’s ABI, Remix IDE lets you copy-paste it  \] 

```

Those two constants are extremely important, we are going to import them in each of the 4 files created above. So, go ahead and include this import at the top of each file:

```

import {ABI , ADDRESS} from "../config" 

```

Also, import web3 to each of the files using:

```

import Web3 from "web3";

````
## Writing the UI - building the interface (App.js):
[https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/App.js](https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/App.js) 

Now, import the modules in the components folder to App.js: 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/00999415-e68b-4844-bbeb-c669abe05fa3.jpg)

Notice the constructor above, it initializes the state object.

Now, write your render() function:

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/987d4af9-72f1-4b7f-9631-567bc7a1c1c6.jpg)

Ok, so we have to tell the player on what network he is playing, so include this web3 script that gets the network type:

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/8b9fe8b2-4fa2-40c0-9f2e-39d0f48056ed.jpg)

What the snippet above is doing is basically creating a web3 object based on the web3 provider in your environment and then fetching the network type.

Cool, now we only have to code our components imported in App.js. Let’s move on!

Subquest: Writing the UI - populating components (GameStatus.js):

[https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/components/GameStatus.js](https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/components/GameStatus.js) 

Ok, we are finally here! Now we have to write components’ codes. As I am assuming a basic knowledge of react.js, I am not going to explain line by line. Instead, I am going to explain web3 scripts and their role in connecting to our contract. So, in GameStatus.js, we have a checkStatus() function that “talks” to our contract specified by ABI and ADDRESS. It just queries to find out if the game is done or not and if so, updates the state. This function is called each time a player hits the game status button, this is handled in the code using handleSubmit() function.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/a4e00098-aa5f-4e3a-a659-bf3e056fa654.jpg)

Remember our GameOver event in the contract? Now is the right time to use it. This is done in the isItOver() function down below. It just creates an instance of the contract and listens for the event. 

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/281537a4-2c96-4d08-9ecc-b417d9690ae4.jpg)

 Great, now we are up to date with the game status. Now let's do the same thing to get turns, i.e. a button that gives back whose turn it is.
## Writing the UI - populating components (Turn.js):
[https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/components/Turn.js](https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/components/Turn.js) 

Like we did in GameStatus.js, we create a button that invokes a function that takes to our contract. This function is checkTurn() shown below. This chain of JavaScript again gets the accounts to let the user communicate with the contract ( to assign his/her address as msg.sender). After that, it calls the getTurn() function from the contract and updates the state.

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/dbcc7d06-10eb-48a4-9395-dd17d89218c4.jpg)
## Writing the UI - populating components (RegisterFrom.js and ShootButton.js):
[https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/components/RegisterForm.js](https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/components/RegisterForm.js) 

[https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/components/ShootButton.js](https://github.com/TheLedgerOfJoudi/russian-roulette/blob/master/src/components/ShootButton.js) 

We are nearly done here, we just need to add a register function and, of course, shooting.

In RegisterForm.js, add a button that calls the function below after clicking, note that this function calls register() from our accounts with a msg.value of 0.5 ethers (expressed in Wei):

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/9b80dd8c-1666-4608-a629-9c06b29d7439.jpg)

The same pattern applies for ShootButton.js, create a button that calls the function shown below. It calls the shoot() function in the contract and alerts the user if anything goes wrong. Seriously, that is the funniest alert I have ever written:

![](https://qb-content-staging.s3.ap-south-1.amazonaws.com/public/fb231f7d-06af-4aff-bca3-fd51cb633f77/3a2c9bff-105c-428f-a3f0-b979fe038661.jpg)
## So what now?
Now run the app using `yarn start`. Open the app in the browser with `localhost:3000` and connect your metamask wallet to play this game on the app :)

Now you created a game on Ethereum! Please feel free to brag about it.

In this quest, you learned how to write a Russian Roulette dapp, the same steps apply to a lot of the dapps you are going to create. Firstly, you write the contract. Secondly, Deploy to get the abi and address. Finally, build the UI and link them together. That is it!  
Happy coding!