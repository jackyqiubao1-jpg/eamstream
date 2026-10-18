---
title: "The Latest Ethereum Solidity Smart Contract Development Tutorial (Part -2) — Develop solidity"
date: 2019-04-04
tags: [ethereum, solidity, blockchain]
draft: false
---


On the previous tutorial [(Part -1) — Solidity Smart Contract development environment setup for Visual Studio Code with Truffle framework](https://medium.com/@jackyqiubao/the-latest-ethereum-solidity-smart-contract-development-tutorial-part-1-develop-solidity-4ee15bdb1dee), we learned how to setup our development environment and the high level of the requirement for rock-scissors-paper game which we are planning to build smart contract for. Now in this tutorial, we will start to write a solidity smart contract and test it in truffle.

Before you start the following section, please make sure you follow the [Part -1](https://medium.com/@jackyqiubao/the-latest-ethereum-solidity-smart-contract-development-tutorial-part-1-develop-solidity-4ee15bdb1dee)to complete development environment setup.

And to be able to have better understand each step and the code in the following, I would like to encourage you to walk through the basic concept on Solidity and Truffle through the below official sites . Of course you would be able to walk through the following steps without knowledge of Truffle and Solidity, but you would only know how to do it, instead of why to it (I think this is more important than how).

(1) Solidity: [https://solidity.readthedocs.io/en/develop/index.html](https://solidity.readthedocs.io/en/develop/index.html)

(2) Truffle: [https://truffleframework.com/docs/truffle/overview](https://truffleframework.com/docs/truffle/overview)

In the VS Code, open the folder which we create for the DAPP, for my case, it is C:\DAPPTutorial. The folder currently is empty.

![](https://cdn-images-1.medium.com/max/800/1*eJREQaYSmpx4b2ODnQqCSg.png)

Now we are ready to start our exciting solidity smart contract coding.

**Step-1:** Create a bare Truffle project.

Truffle provided some of example applications and project templates at [Truffle Boxes](https://www.truffleframework.com/boxes) which you can pull them to your local through truffle unbox command. But for our tutorial, I would like to start from a blank project template to let you have better understanding on the folder structure of Truffle.

Open a new terminal in VS Code, make sure the current folder is at the C:\DAPPTutorial.

![](https://cdn-images-1.medium.com/max/800/1*ojJGiwn6M5YR2Rviev4X4Q.png)

Type the following command and enter on this terminal:

*truffle init*

After this command execute, you will find there are couple files and folders are created under the folder of DAPPTutorial.

![](https://cdn-images-1.medium.com/max/800/1*_NPo1IxX6HWuNI1WvfJVnA.png)

Let’s have a quick walk-through of this folder and file structure:

(1) Folder “contracts ”— this folder is for smart contract solidity file.

(2) File “Migrations.sol” under contracts folder — Truffle migrations enable us to “push” the smart contracts to the Ethereum blockchain (either local, tesnet or mainnet) and to set up necessary steps for linking contracts with other contracts as well as populate contracts with initial data. The `Migrations.sol` file can look any way you want it to, but it must conform to a fixed interface which looks like the interface created by the `truffle init` command. What you can do in those files is some advanced mangling of migrations, but it’s rarely needed.

(3) Folder “migrations” — this folder is used to manage deployment script.

(4) File “1_initial_migration.js” under migration folder— The same goes for the `1_initial_migration.js` file. What *it* does is simply push the `Migrations.sol` file to the desired blockchain.

(5) Folder “test” — this folder is for the files for smart contract testing purpose.

(6)File “truffle-config.js” — This file is a Javascript file and can execute any code necessary to create your configuration. We will cover more detail on this file in the next step.

**Step-2:** Modify truffle-config.js to connect to ganache

As mentioned before, the file truffle-config.js is used for the truffle configuration. Uncomment the following code snippet by removing “//” in truffle-config.js which will allow us to connect to ganache blockchain we launched locally in Part-1.

> development: {

> host: “127.0.0.1”, // Localhost (default: none)

> port: 8545, // Standard Ethereum port (default: none)

> network_id: “*”, // Any network (default: none)

> },

Make sure you save this file after the change.

**Step-3:** Start the coding for smart contract to meet the requirement of rock-scissors-paper game.

In the VS Code Explorer, click to select “contracts” folder, and click on new file button at the top right.

![](https://cdn-images-1.medium.com/max/800/1*ZG2eD1bFIE8z4x7Pp6xZ3A.png)

In the input box, type “Rockscissorspaper.sol” as our smart contract file name (sol is solidity file).

Let’s have a quick review on the requirement for rock-scissors-paper game smart contract:

(1) Two players for each game, one is host, other one is client.

(2) Anyone can host a rock-scissors-paper game with certain stake and wait for client to join the game.

(3) Player can view all of the hosted games and select to join one of the game as client player with the same stake as the host player.

(4) Once client player join the game, both host player and client player can play the game by providing input of Rock, Scissors or Paper.

(5) After both of host and client players provide the input, the smart contract will reveal the winner and transfer the both of the stakes to the winner after reducing certain percentage of commission.

The following are the code snippets along with the explanation.

> pragma solidity >=0.4.21 <0.6.0;

All of solidity smart contract will start from this pragma to identify which version of the solidity will be compatible for the smart contract.

> contract RockScissorsPaper {

> …

> }

The smart contract name “RockScissorsPaper” which we will create.

> enum BetValue

> { //enum to list all of the values the player can put for the game

> Rock,Scissor,Paper

> }

> enum GameStatus

> { // enum for the status of the game

> Invalid,Hosted,Joined,Finished

> }

> struct Player

> { // structure to store player information

> bytes32 playerName;

> address payable player;

> BetValue playerBetValue;

> bool isPlayed;

> bool isWon;

> }

> struct Game

> { // structure to store game information

> uint gameID;

> Player host;

> Player client;

> GameStatus status;

> uint betAmount;

> }

> //Pending withdrawls amount for the players

> mapping(address => uint) public pendingWithdrawls;

> //The list of the games

> Game[] private gamesList;

> //The address of contract owner who can receive commission for finished game.

> address payable private commissioner;

> //The commission percentage of the stake

> uint8 private commissionPercent;

> //To keep acculated commission amount

> uint private totalCommission;

> //Total number of the hosted games

> uint private hostNumber;

The above code snippet is the data structure of this smart contract. They are used to keep the data of smart contract.

> modifier verifyGame(uint _gameID) {

> require(gamesList.length>_gameID,’The game is not exist’);

> _;

> }

You can think of function modifier as an extension of a function. Modifier code will run first when assigned by a function. “_;” means to return the flow of execution to the original function code. Function modifier is mainly used to check a condition prior to executing the function. So you could re-use the same modifier in multiple functions if you are checking for the same condition. It reduces code redundancy.

> constructor(uint8 _commissionPercent) public

> { // contract constructor

> commissioner = msg.sender;

> commissionPercent=_commissionPercent;

> hostNumber=0;

> }

The constructor will be executed when the smart contract is deployed. Usually we use it to initialize some variables.

> function hostGame(bytes32 _hostName) public payable returns(uint)

> { //host a game and return the game id

> require(msg.value>0,’Bet amount need to more than 0');

> require(_hostName.length>0,’The name cannot empty’);

> Player memory hostPlayer;

> hostPlayer.player=msg.sender;

> hostPlayer.playerName=_hostName;

> hostPlayer.isPlayed=false;

> Game memory game;

> uint gameID;

> gameID=gamesList.length;

> game.gameID=gameID;

> game.host=hostPlayer;

> game.betAmount=msg.value;

> game.status=GameStatus.Hosted;

> gamesList.push(game);

> pendingWithdrawls[msg.sender]+=msg.value;

> hostNumber+=1;

> return gameID;

> }

This function is used to allow the user to host a game and return the new game’s id.

> function registerAsAnOpponent(uint _gameID, bytes32 _clientName) public payable verifyGame(_gameID)

> { // Register hosted game _gameID as oppenent (client player)

> require(gamesList[_gameID].client.player==address(0),’The game was occupied’);

> require(gamesList[_gameID].status==GameStatus.Hosted,’The game status is not allowed to join’);

> require(msg.value==gamesList[_gameID].betAmount,’You need to bet for the same amount as host’);

> require(_clientName.length>0,’The name cannot empty’);

> Player memory clientPlayer;

> clientPlayer.player=msg.sender;

> clientPlayer.playerName=_clientName;

> clientPlayer.isPlayed=false;

> gamesList[_gameID].client=clientPlayer;

> gamesList[_gameID].status=GameStatus.Joined;

> pendingWithdrawls[msg.sender]+=msg.value;

> hostNumber-=1;

> }

This function will allow user to choose a hosted game to join as client player.

> function playGame(uint _gameID, uint8 _playValue) public payable verifyGame(_gameID)

> { // Player provide input (rock, scissors or paper) for the game _gameID

> // Will reveal game result if both of the player are provide the input

> require(gamesList[_gameID].status==GameStatus.Joined,’No one join the game yet’);

> require(msg.sender == gamesList[_gameID].client.player || msg.sender ==gamesList[_gameID].host.player,’You are hacking to the game!!!!!!!!!’);

> require(_playValue>=0 && _playValue<=2, ‘Wrong Bet input’);

> if(msg.sender == gamesList[_gameID].host.player)

> {

> if(!gamesList[_gameID].host.isPlayed)

> {

> gamesList[_gameID].host.isPlayed = true;

> gamesList[_gameID].host.playerBetValue=BetValue(_playValue);

> }

> } else

> {

> if(!gamesList[_gameID].client.isPlayed)

> {

> gamesList[_gameID].client.isPlayed = true;

> gamesList[_gameID].client.playerBetValue=BetValue(_playValue);

> }

> }

> if(gamesList[_gameID].host.isPlayed && gamesList[_gameID].client.isPlayed)

> {

> int betResult=checkBetResult(gamesList[_gameID].host.playerBetValue,gamesList[_gameID].client.playerBetValue);

> if(betResult==1) //host win

> {

> gamesList[_gameID].host.isWon=true;

> gamesList[_gameID].client.isWon=false;

> endOfGame(_gameID,gamesList[_gameID].host.player,gamesList[_gameID].client.player);

> } else if (betResult==-1) //client win

> {

> gamesList[_gameID].host.isWon=false;

> gamesList[_gameID].client.isWon=true;

> endOfGame(_gameID,gamesList[_gameID].client.player,gamesList[_gameID].host.player);

> } else // equal

> {

> gamesList[_gameID].host.isPlayed=false;

> gamesList[_gameID].client.isPlayed=false;

> }

> }

> }

The playGame function will reveal game result if both of the player are provide the input.

> function getAvailableHostList() public view returns (bytes32[] memory,address[] memory, uint[] memory, uint[] memory)

> {

> bytes32[] memory hostNames=new bytes32[](hostNumber);

> address[] memory hostAddresses=new address[](hostNumber);

> uint[] memory hostIDs=new uint[](hostNumber);

> uint[] memory betAmounts=new uint[](hostNumber);

> uint hostIndex=0;

> for (uint index = 0; index < gamesList.length; index++)

> {

> Game memory game=gamesList[index];

> if(game.status==GameStatus.Hosted)

> {

> hostNames[hostIndex]=game.host.playerName;

> hostAddresses[hostIndex]=game.host.player;

> betAmounts[hostIndex]=game.betAmount;

> hostIDs[hostIndex]=index;

> hostIndex+=1;

> }

> }

> return (hostNames,hostAddresses,hostIDs,betAmounts);

> }

This function will expose external application (for example, javascript) to get all of the available hosted game list.

---
