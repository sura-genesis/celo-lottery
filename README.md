# Building a Decentralized Lottery on CELO Blockchain
## Building a Decentralized Lottery on the CELO Blockchain: From Smart Contract Development to Deployment

Decentralized applications (DApps) have gained significant popularity in the blockchain space, offering transparency, security, and immutability. Smart contracts, self-executing contracts with coded terms, play a crucial role in powering DApps.

This article guides you through the process of building and deploying a decentralized lottery smart contract on the CELO blockchain. CELO, a blockchain platform, facilitates fast and cost-effective transactions, making it an excellent choice for decentralized applications.

## Table of Contents

- [Section 1: Understanding the Basics](#understanding-the-basics)
  - [1.1 Overview of Lottery Smart Contracts](#overview-of-lottery-smart-contracts)
- [Section 2: Code Explanation of the Smart Contract](#section-2-code-explanation-of-the-smart-contracts)
  - [2.1 Key Variables](#21-key-variables)
  - [2.2 Event](#22-event)
  - [2.3 Modifiers](#23-modifiers)
  - [2.4 Constructor](#24-constructor)
  - [2.5 Managing Rounds and Selecting Winners](#25-managing-rounds-and-selecting-winners)
    - [2.5.1 Entering a Round](#251-entering-a-round)
    - [2.5.2 Starting a New Round](#252-starting-a-new-round)
    - [2.5.3 Ending the Current Round](#253-ending-the-current-round)
  - [2.6 Checking Round Status and Participants](#26-checking-round-status-and-participants)
    - [2.6.1 Getting Current Round Status](#261-getting-current-round-status)
    - [2.6.2 Generating a Random Number](#262-generating-a-random-number)
    - [2.6.3 Retrieving Players](#263-retrieving-players)
- [Section 3: Complete Code](#complete-code)
- [Section 4: Getting Celo faucet](#getting-celo-faucet)
- [Section 5: Deployment to remix](#deployment-to-remix)
- [Section 6: Conclusion](#section-6-conclusion)
  - [5.1 Next Steps](#51-next-steps)


## Section 1: Understanding the Basics

### 1.1 Overview of Lottery Smart Contracts
Decentralized lottery smart contracts bring transparency and fairness to the process of organizing and participating in lotteries. This guide explores the fundamental concepts and components involved in building such a contract.

## Section 2: Code Explanation of the Smart Contract

### 2.1 Key Variables
```solidity
    address public manager;
    address[] public players;
    address public lastWinner;
    uint256 public roundEndTime;
    uint256 public minimumPlayers;
    bool public isRoundActive;
```
These variables represent essential components of the lottery smart contract. The manager initiates the lottery, players contribute funds to participate, lastWinner keeps track of the previous winner, roundEndTime determines when a round ends, minimumPlayers sets the threshold for starting a new round, and isRoundActive indicates whether a round is currently active.

### 2.2 Event
```solidity
event LotteryWinner(address winner, uint256 amount);
```
When this event is emitted within the smart contract, it records the winner's address and the corresponding prize amount. This logged information can be observed by external applications or user interfaces, enabling them to track and display details about lottery winners and prize amounts on the blockchain.

### 2.3 Modifiers
```solidity
 modifier restricted() {
        require(msg.sender == manager, "Only the manager can call this function");
        _;
    }

    modifier roundActive() {
        require(isRoundActive, "The current round is not active");
        _;
    }
```
The `restricted` modifier ensures that only the manager can call certain functions, providing access control. The `roundActive` modifier checks whether the current round is active before allowing certain operations.

### 2.4 Constructor
```solidity
   constructor(uint256 _minimumPlayers) {
        manager = msg.sender;
        minimumPlayers = _minimumPlayers;
        isRoundActive = false;
    }
```
The constructor function initializes the decentralized lottery smart contract by setting the contract manager to the deployer's address (msg.sender), defining the minimum required players (_minimumPlayers), and initially marking the round as inactive (isRoundActive = false).

### 2.5 Managing Rounds and Selecting Winners
  ### 2.5.1 Entering a Round
  ```solidity
    function enter() public payable {
        require(msg.value > 0.01 ether, "Minimum contribution is 0.01 ether");
        require(!isRoundActive, "Cannot enter while a round is active");
        
        players.push(msg.sender);
    }
  ```
The enter function allows players to participate in the lottery by contributing funds. It imposes a minimum contribution requirement and ensures that entries are not allowed during an active round.

  ### 2.5.2 Starting a New Round
  ```solidity   
    function startNewRound() public restricted {
        require(!isRoundActive, "A round is already active");

        // Ensure there are enough players to start a round
        require(players.length >= minimumPlayers, "Not enough players to start a round");

        roundEndTime = block.timestamp + 24 hours; // Round lasts for 24 hours
        isRoundActive = true;
    }
  ```
The `startNewRound` function is responsible for initiating a new lottery round. It resets the round-related variables, sets a new round end time, and marks the round as active.

  ### 2.5.3 Ending the Current Round
  ```solidity
    function endRound() public restricted roundActive {
        require(block.timestamp >= roundEndTime, "Round is still active");
        // Pick a winner only if there are enough players
        if (players.length > 0) {
            uint256 index = random() % players.length;
            lastWinner = players[index];

            // Emit winner event
            emit LotteryWinner(lastWinner, address(this).balance);

            // Transfer the entire contract balance to the winner
            payable(lastWinner).transfer(address(this).balance);
        }
  ```
The `endRound` function is responsible for finalizing the current round, selecting a winner randomly, transferring the winnings to the winner, and resetting the players array and round status for the next round.

### 2.6 Checking Round Status and Participants
  ### 2.6.1 Getting Current Round Status
  ```solidity
   function getCurrentRoundStatus() public view returns (bool active, uint256 endTime) {
        return (isRoundActive, roundEndTime);
    }
   ```
This function query the contract and obtain information about the current round's status and end time.

  ### 2.6.2 Generating a Random Number
```solidity
function random() private view returns (uint256) {
        return uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp, players)));
    }
  ```
  This function is a private view function designed to generate a pseudo-random number.
  
  ### 2.6.3 Retrieving Players
  ```solidity
  function getPlayers() public view returns (address[] memory) {
        return players;
    }
  ```
 This function retrieves the array of participant addresses (players) in the current lottery round.
 
### Section 3: Complete Code
```solidity
// 

SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract Lottery {
    address public manager;
    address[] public players;
    address public lastWinner;
    uint256 public roundEndTime;
    uint256 public minimumPlayers;
    bool public isRoundActive;

    event LotteryWinner(address winner, uint256 amount);

    modifier restricted() {
        require(msg.sender == manager, "Only the manager can call this function");
        _;
    }

    modifier roundActive() {
        require(isRoundActive, "The current round is not active");
        _;
    }

    constructor(uint256 _minimumPlayers) {
        manager = msg.sender;
        minimumPlayers = _minimumPlayers;
        isRoundActive = false;
    }

    function enter() public payable {
        require(msg.value > 0.01 ether, "Minimum contribution is 0.01 ether");
        require(!isRoundActive, "Cannot enter while a round is active");
        
        players.push(msg.sender);
    }

    function startNewRound() public restricted {
        require(!isRoundActive, "A round is already active");

        // Ensure there are enough players to start a round
        require(players.length >= minimumPlayers, "Not enough players to start a round");

        roundEndTime = block.timestamp + 24 hours; // Round lasts for 24 hours
        isRoundActive = true;
    }

    function endRound() public restricted roundActive {
        require(block.timestamp >= roundEndTime, "Round is still active");

        // Pick a winner only if there are enough players
        if (players.length > 0) {
            uint256 index = random() % players.length;
            lastWinner = players[index];

            // Emit winner event
            emit LotteryWinner(lastWinner, address(this).balance);

            // Transfer the entire contract balance to the winner
            payable(lastWinner).transfer(address(this).balance);
        }

        // Reset the players array and round status for the next round
        players = new address[](0);
        isRoundActive = false;
    }

    function getPlayers() public view returns (address[] memory) {
        return players;
    }

    function getCurrentRoundStatus() public view returns (bool active, uint256 endTime) {
        return (isRoundActive, roundEndTime);
    }

    function random() private view returns (uint256) {
        return uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp, players)));
    }
}

```
### Section 4: Getting Celo faucet 
The Celo Faucet is a tool that allows developers to obtain testnet CELO tokens for development and testing purposes on CELO blockchain.

Visit Celo Faucet: [celo faucet]:https://faucet.celo.org/alfajores
Paste your wallet address. You can authenticate with your Github to get 10x of the token.
![celoFaucet](https://github.com/bodmandao/celo_decentralized_lottery/assets/154741685/d0ac677e-e78c-4c89-a888-6f06a6ebce2d)

Click on faucet to get your token.

### Section 5: Deployment to remix
Go to remix[]:https://remix.ethereum.org/

Create a new file named `lottery.sol` and paste the complete the code.
![remix](https://github.com/bodmandao/celo_decentralized_lottery/assets/154741685/2168e375-cdbb-4f3d-b3d8-aff239b9d148)

Compile the lottery contract.

Don't forget to configure `celo alfajores testnet` on your metamask. You can refer to this documentation [Link]:https://celo.academy/t/3-simple-steps-to-connect-your-metamask-wallet-to-celo/84

Under the `environment` tab, select `injected provider` to connect to metamask or any equivalent.

Set `minimum player` and click on `deploy` button on remix after you have successfully connected your wallet.

Congratulations! You have successfully deployed a decentralized on CELO blockchain. You can go ahead to interact with your contract.
![deployed](https://github.com/bodmandao/celo_decentralized_lottery/assets/154741685/8b2c461a-c6a6-48c5-b7df-f5c66b2d1fc0)



