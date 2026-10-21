---
title: "The Latest Ethereum Solidity Smart Contract Development Tutorial (Part -3) — Compile, Deploy and"
date: 2019-04-11
tags: [ethereum, solidity, blockchain]
draft: false
---


If you following my previous 2 serials:

[(Part -1) — Solidity Smart Contract development environment setup for Visual Studio Code with Truffle framework](/posts/ethereum-solidity-tutorial-part1-develop/)

[(Part -2) — Develop solidity smart contract in truffle](/posts/ethereum-solidity-tutorial-part2/)

you should be able to setup your solidity smart contract development environment and write your first smart contract. Now let’s take further adventure to compile, deploy and test the smart contract we developed in previous tutorial.

**Step 1 **— Compile smart contract

In a VS Code terminal window, make sure you are at the current DAPP folder, in our case, it is C:\DAPPTutorial. Type the command “truffle compile” and enter, you will get the result like this:

![](/images/1*bZonNxlEAIsJmhWLo6b5Rw.png)

You should find some json files is created under the folder .\build\contracts

![](/images/1*bObogKeJ4AF0dlyEUjW8GQ.png)

Those `.json` file are the ABI (Application Binary Interface) that describe the deployed contract and its functions. It allows us to contextualize the contract and call its functions, which will allow external application, for example, web3.js to call the exposed functions for the deployed smart contract. We will learn more detail on it on next serial Part-4 How to call smart contract from web javascript.

**Step 2**— Create migration script in folder migrations

Migrations are JavaScript files that help you deploy contracts to the Ethereum network. These files are responsible for staging your deployment tasks, and they’re written under the assumption that your deployment needs will change over time. As your project evolves, you’ll create new migration scripts to further this evolution on the blockchain. A history of previously run migrations is recorded on-chain through a special Migrations contract.

Create a new file call “2_deploy_RockScissorsPaper.js” under migration folders.

![](/images/1*xl91QgX55yYCE-yr4-L9EQ.png)

Note that the filename is prefixed with a number and is suffixed by a description. The numbered prefix is required in order to record whether the migration ran successfully. The suffix is purely for human readability and comprehension.

Open this new created “2_deploy_RockScissorsPaper.js” file in VS Code and type the following code to it.

> var MyContract = artifacts.require(“RockScissorsPaper”);

> module.exports = function(deployer) {

> // deployment steps

> deployer.deploy(MyContract,5);

> };

In the above script, it is used to deployed our smart contract “RockScissorsPaper”. The script number 5 in the statement *deployer.deploy(MyContract,5) *is the parameter to pass to the contract construction function:

> constructor(uint8 _commissionPercent) public

> { // contract constructor

> commissioner = msg.sender;

> commissionPercent=_commissionPercent;

> hostNumber=0;

> }

Save this js file and we will move to next step to migrate (deploy) this contract to ganache.

**Step 3** — Migrate (Deploy) smart contract to ganache

Make sure the ganache-cli is running, and in the VS Code terminate, run the following command (please make sure in that terminate you are under current application folder) :

> truffle migrate

You will see the following similar screen after the command being executed:

![](/images/1*LNfeK32L0dLbYM_75fnDoA.png)

The first smart contract deployment is for the Migrations.sol which is to support truffle framework. The second contract deployment is for our Rockscissorspaper.sol smart contract. And from the result, we can get contract address, and how many gas cost to deploy the contract.

After the contract deployed, the *constructor *function for the contract is executed, and it is the only chance to run this constructor.

Now we deployed our smart contract to the blockchain. Next we will see how to call the mart contract functions through truffle console.

Step 4 — Call smart contract function in truffle console

In a VS Code terminate, run the following command to launch truffle console:

> truffle console

Then you will see we are in the truffle console as following:

![](/images/1*uT6aRJcV46NMqj4kqA7Bbg.png)

You can quit from the truffle console by execute the command *.exit *in the truffle console to quit from truffle console.

In the truffle console, let’s try to call *hostGame function in the *Rockscissorspaper smart contract by execute the following command:

> RockScissorsPaper.deployed().then(function(instance){return instance.hostGame( web3.utils.asciiToHex(‘Host’),{from:’0x15b7b19813454ba6904d6b233a6505fbdbc13e28',value:web3.utils.toWei(‘1’, ‘ether’)})})

You might be curious in our hostGame function, only require one parameter *bytes32* _hostName. Why in this command, looks like we pass 3 parameters. Actually the following section in the command is to identify account to call this function and how much Ether we will pass to this function, since this function is payable.*{from:’0x15b7b19813454ba6904d6b233a6505fbdbc13e28',value:web3.utils.toWei(‘1’, ‘ether’)}*

Note: *The account address you can get from ganache-cli console when you launch ganache-cli.*

After you execute this command, you should be able to see the similar result as following:

![](/images/1*6gLvzfkeY1CX_A_RpRg1JA.png)

Now let call getAvailableHostList function to see if the account *’0x15b7b19813454ba6904d6b233a6505fbdbc13e28' *hosted the game.

Execute the following command in truffle console:

> RockScissorsPaper.deployed().then(function(instance){return instance.getAvailableHostList();})

We will see the following result as we expect.

![](/images/1*puT3XaogFxE1sPGtDFcYmA.png)

It is time for other user to call the function registerAsAnOpponent to join this hosted game.

> RockScissorsPaper.deployed().then(function(instance){return instance.registerAsAnOpponent(0,web3.utils.asciiToHex(‘client’),{from:’0xd533b3651588072cf662814434976d0375b395e5',value:web3.utils.toWei(‘1’, ‘ether’)})})

The result should be similar as following:

![](/images/1*uDLOXFbosYp80G3yZNv97g.png)

Let start to play the game. The host “0x15b7b19813454ba6904d6b233a6505fbdbc13e28” will play scissors (1) by the following command:

> RockScissorsPaper.deployed().then(function(instance){return instance.playGame(0,1,{from:’0x15b7b19813454ba6904d6b233a6505fbdbc13e28'})})

The result will be similar as:

![](/images/1*QcrBuPME9DYc5_2VD7wtVg.png)

Now it is time for client “0xd533b3651588072cf662814434976d0375b395e5” to play. The client would like to play paper (2):

> RockScissorsPaper.deployed().then(function(instance){return instance.playGame(0,2,{from:’0xd533b3651588072cf662814434976d0375b395e5'})})

After both of players play the game, let’s call getGameInfo to check who is winner:

> RockScissorsPaper.deployed().then(function(instance){return instance.getGameInfo(0)})

Per the result, we can see the host win.

![](/images/1*flbZ4CGlIsaNbfoLzDDwzg.png)

Congratulation! Now you know how to write the smart contract, and how to deploy and test it in truffle if you follow this serial tutorials from part -1 to part -3.

In the last article Part -4 , I will show you how to call solidity smart contract from web through web3.js. You cannot expect your users to use truffle to play Rock_Scissors _Paper game, right?:)

---
