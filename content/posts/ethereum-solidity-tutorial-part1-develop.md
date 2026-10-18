---
title: "The Latest Ethereum Solidity Smart Contract Development Tutorial (Part -1) — Develop Solidity"
date: 2019-04-02
tags: [ethereum, solidity, blockchain]
draft: false
---


There is blossomed increasing blockchain development skills resource request in the job market in the last year (2018), and this trend is continuing in 2019. To increase your job opportunities if you are willing to embrace blockchain revolution, learning popular smart contract development technology such as solidity will arm yourself.

Thanks for many of selfless writers, there are a lot tutorials you can find through google. But since blockchain technology is still evolving, the related development tools / languages keep updating in months. If you are follow out of date articles to start your smart contract learning, I am pretty sure you will lose a lot of your hairs :). To save your hair, I write these serial tutorials to guide you how to develop smart contract in Solidity for a real use case, and more than that, you can learn some of the key technologies related to smart contract development, and the whole software development life cycle from requirement to production. You can get benefit from these articles whether you are developer or technical manager. Here are the highlights you will learn from these serial articles:

- Basic concept of Ethereum Smart Contract.
- Hand-on experiment on Develop a real world smart contact use case in integral process from requirement to implementation in solidity language based on Ethereum blockchain: Rock Scissors Paper game
- Key points of the latest version solidity.
- The edge technologies or tools (such as Truffle, Visual Studio Code, GIT) along with blockchain to build entire smart contract solution.

These serial tutorial is suitable to the readers who have basic concept of Ethereum blockchain, and the basic skill or hand-on experience on development with at least one language.

The following are of the serial tutorials I am planning to write:

(1) (Part -1) — Solidity Smart Contract development environment setup for Visual Studio Code with Truffle framework

(2) (Part -2) —Develop solidity smart contract in Truffle

(3) (Part -3) — Compile, Deploy and Test solidity smart contract in Truffle

(4) (Part -4) — Build Frontend html page with java script to connect to solidity smart contract ABI through Web3 and MetaMask

This article is for Part -1. I will publish part 2 , 3 & 4 in the near future.

Before we are getting start on technical section, let’s take quick look on what’s the requirement of this use case and why we need to use smart contract blockchain to solve this problem.

I think most of the people know how to play rock-scissors-paper game (for the folks don’t know this game, please refer to wiki: [https://en.wikipedia.org/wiki/Rock%E2%80%93paper%E2%80%93scissors](https://en.wikipedia.org/wiki/Rock%E2%80%93paper%E2%80%93scissors)).

![](https://cdn-images-1.medium.com/max/800/1*JyseWEl7Aw5RlHqwd0sb5A.png)

*Rock-Scissors-Paper Game*

Assume that you want to build a website to allow people to play this game. But how can you make sure your users trust you the website will not cheat them? Since it is impossible for both of the players to provide the input (Rock, Scissors or Paper) at the same time through internet, how can we build a solution to reveal the result base on the input from both of the players without being cheated (for example, the owner of the website can peep the competitor’s input). Yes, blockchain is the right solution to solve these trust problem. That is why most of the current DAPPs are occupied by gambling games in Ethereum and EOS ( of course DAPP is not the only purpose for gambling). Here are the key requirements / features we need to build a smart contract for this rock-scissors-paper game:

(1) Two players for each game, one is host, other one is client.

(2) Anyone can host a rock-scissors-paper game with certain stake and wait for client to join the game.

(3) Player can view all of the hosted games and select to join one of the game as client player with the same stake as the host player.

(4) Once client player join the game, both host player and client player can play the game by providing input of Rock, Scissors or Paper.

(5) After both of host and client players provide the input, the smart contract will reveal the winner and transfer the both of the stakes to the winner after reducing certain percentage of commission.

All right, now we have requirement and good to start solidity smart contract coding. Hold on second, before we can really code, we need to setup our development environment. Let take an overview of technical stacks we gonna to use in this tutorial:

OS: Windows 10

IDE: Visual Studio Code 1.32.3

Tools / Packages:

(1) Node.js 10.7.0 and NPM 6.1.0 (javascript package management)

npm is included as a recommended feature in Node.js installer. npm consists of a command line client that interacts with a remote registry. It allows users to consume and distribute JavaScript modules that are available on the registry.

(2) Truffle v5.0.5

Truffle is a development environment, testing framework and asset pipeline for Ethereum, aiming to make life as an Ethereum developer easier. It is one of the most widely used IDEs in the Ethereum community. Developers can use it to build and deploy DApps for testing purposes with many features that make it more attractive to users with a Web 3.0 dev background. It will include solidity language which is the language we are using for Ethereum smart contract development.

(3) Ganache-cli v6.4.0

Ganache CLI is the latest version of TestRPC: a fast and customizable blockchain emulator. It allows you to make calls to the blockchain without the overheads of running an actual Ethereum node.

Please follow the below sequence to install and config the required tools for development environment. If your OS is different with Windows, please follow the installation instruction relate to your OS on the tools / packages website.

- Install Node.js and npm

Visit node.js website to download node.js and npm install package:

Select your OS related install package to download. Let’s choose Windows Installer (.msi) 64bit version to download and install it.

2. Install Visual Studio Code (VS Code)

VS Code is a lite weight source code editor developed by Microsoft for Windows, Linux and macOS. It includes support for debugging, embedded Git control, syntax highlighting, intelligent code completion, snippets, and code refactoring. It can include new feature through install extensions.

Download VS Code from the below website for your OS: [https://code.visualstudio.com/download](https://code.visualstudio.com/download)

In my case, I download windows user installer 64 bit version. Install Visual Studio Code after download install file.

3. Setup project folder in VS Code

Open VS Code, click on “Open folder…” menu on the welcome window.

![](https://cdn-images-1.medium.com/max/800/1*M7wzkwDL9uEv-wZR4YaHwg.png)

Create a new folder call “DAPPTutorial” folder under C drive and click select this folder button.

![](https://cdn-images-1.medium.com/max/800/1*xqCoQddfPhwazLwbdPUf6w.png)

4. Install ganache-cli

To install ganache-cli, you can use npm through command prompt window, or through terminal in VS Code. I will install it through VS Code terminal (for the rest of other javascript packages, if I don’t specify, will be installed through VS Code terminal).

Open a new terminal through menu Terminal -> New Terminal

![](https://cdn-images-1.medium.com/max/800/1*1zIym5BAotL1TCY-7y6oMg.png)

On the terminal window, type the following command and enter:

*npm install -g ganache-cli*

![](https://cdn-images-1.medium.com/max/800/1*KskqOSyaJeG8sJ_pcsv8LQ.png)

Notes: -g option is to install ganache-cli package globally in your current npm environment.

After install successfully, type the following command to start ganache:

*ganache-cli*

It will launch a local Ethereum blockchain along with 10 accounts for development testing. And open a default local port 8545 for transaction.

![](https://cdn-images-1.medium.com/max/800/1*rlYyN1e7CaoM4QpiblBLXQ.png)

5. Install Truffle

Open a new split terminal through click split window button on the right top of terminal window.

![](https://cdn-images-1.medium.com/max/800/1*Hz0HIPFIglSurCnmnkCL6A.png)

On the new terminal window, type the following command and enter to install truffle:

*npm install -g ganache-cli*

After install truffle, type the following command to see the version of truffle and related packages:

*truffle version*

![](https://cdn-images-1.medium.com/max/800/1*o6T4bcoJ3dW3iqR89XKBWA.png)

Congratulation, now your smart contract development environment is ready. I will walk your through on how to develop solidity smart contract to fulfill the requirement for the Rock-Scissors-Paper game in the next tutorial:

[(Part -2) — Develop solidity smart contract in Truffle](https://medium.com/@jackyqiubao/the-latest-ethereum-solidity-smart-contract-development-tutorial-part-2-develop-solidity-871f743ad113)

Please let me know if you have any issue / question on this tutorial.

Please don’t

---
