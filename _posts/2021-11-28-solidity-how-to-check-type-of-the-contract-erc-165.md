---
layout: post
date: 2021-11-28
title: How to check the type of the contract in Solidity - ERC165 explained
categories: [crypto, coding, solidity]
image: /images/2021-11-28/interface.jpg
---

Solidity does not have the concept of `typeof` or `isinstance`. Given a contract address you cannot find out if this is of a particular contract. But there is a way to do it if contract has implemented the `ERC165`  standard.

`ERC165` standard has just one method
```solidity
function supportsInterface(interfaceId) public view returns (bool);
```

This method takes the interface id as the param and checks if the contract supports that interface. 
What does this mean? Solidity has in built method called `type` which returns the unique id of the interface. The interface you will supply here is of the contract whose type you want to check in another contract.
```solidity
type(MyInterface).interfaceId
```

<!--more-->

OpenZepplin has created a utility contract called `ERC165Storage` which implements the `ERC165` standard and also has a method using which you can define which interfaces does the contract supports.
```solidity
function _registerInterface(bytes4 interfaceId) internal;
```

This method can be supplied an interface id which the contract will save. If the same interface is queried for support it will return true.

OpenZepplin has created `ERC165Checker` to call `supportsInterface` on an address.
```solidity
function supportsInterface(address account, bytes4 interfaceId) internal view returns (bool);
```

This method calls `supportsInterface` on the address to check if the address supports the given `interfaceId`.

How to use this?
This is useful when you want to check if the given address is of a particular contract. But you should know before hand that the given address has implemented `ERC165`. Here is an example how you will use this.

The below contract inherits `ERC165` and calls `_registerInterface` method in the constructor.

```solidity
contract Sum is ERC165Storage, ISum {

	constructor {
		_registerInterface(type(ISum).interfaceId);
	}

	function sum(uint256 a, uint256 b) public override returns (uint256) {
		return a + b;
	}

}
```

The below contract wants to check if the given address is of particular contract type.
```solidity
contract Checker is ERC165Checker {
	
	function sumOrZero(address _add, uint256 a, uint256 b) public view returns (uint256) {
		uint256 answer;
		if (supportsInterface(_add, type(ISum).interfaceId)) {
			answer = ISum(_add).sum(a, b);
		}
		return answer;
	}

}
```

This is useful when you want to restrict what kind of contracts that can be supplied to the function. Each contract you want to allow for needs to implement `ERC165` .