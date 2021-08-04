---
layout: post
date: 2021-08-04
title: How to use call for contract function calls and payments in Solidity
categories: [crypto, coding]
image: /images/2021-08-04/calls.jpg
---

Solidity has the `call` function on `address` data type which can be used to call public and external functions on contracts. It can also be used to transfer ether to addresses.

`call` is not recommended in most situations for contract function calls because it bypasses type checking, function existence check, and argument packing. It is preferred to import the interface of the contract to call functions on it.

`call` is used to call the `fallback` and `receive` functions of the contract. Receive is called when no data is sent in the function call and ether is sent. Fallback function is called when no function signature matches the call.

`call` consumes less gas than calling the function on the contract instance. So in some cases `call` is preferred for gas optimization.

<!--more-->

## How to use call method?
To use `call` you need to send encoded data as the param. The data will have the function signature and params encoded together.

```solidity

function myFunction(uint _x, address _addr) public returns(uint, uint) {
    // do something
	  return (a, b);
}

// function signature string should not have any spaces
(bool success, bytes memory result) = addr.call(abi.encodeWithSignature("myFunction(uint,address)", 10, msg.sender);
```

The return value for the `call` function is a tuple of a boolean and bytes array. The boolean is the success or failure status of the call and the bytes array has the return values of the contract function called which need to be decoded.

To get the value of result in the actual data types you have to use `abi.decode`

```solidity
(uint a, uint b) = abi.decode(result, (uint, uint));
```

decode method takes in the bytes array returned from the function call and the tuple of data types. The returned values are in the same order as the data types tuple.

The 	`call` method does not throw any error and the code execution continues. You have to check for success status yourself and do the needful task on success or failure.

If the function passed to the `call` method does not exist the `fallback` function of the contract is called. If the `fallback` function is not implemented in the contract the `call` method will return success status as false.

## How to specify gas and transfer amount
It is possible to supply amount of gas and ether to the `call` method.
```solidity
_addr.call{value: 1 ether, gas: 1000000}("myFunction(uint,address)");
```

To call the `myFunction` method on the contract only the specified amount of gas will be supplied. The amount of the gas supplied to the outer function where `call` is executed has to be more or equal to the gas supplied to the `call`.

If no `gas` is mentioned all the gas remaining before the call is supplied to the `myFunction` call. It is always better to not hardcode the gas supplied to the function call.

If you are sending ether to the `call`  method,  `myFunction`  should be a `payable` function. Otherwise the call will fail.

If the amount of gas supplied to `call` is less than required by `myFunction` to execute completely the call will fail.

## How to send ether using call
After the Istanbul hardfork  `send` and `transfer` methods have been deprecated. Istanbul hardfork increased the gas cost for `SLOAD` opcode which is used to read a word from the blockchain. This has been explained in [this](https://consensys.net/diligence/blog/2019/09/stop-using-soliditys-transfer-now/) post by Consensys.

If `transfer`  is used inside a function call of a contract a fixed gas of 2300 is supplied to the `transfer` method. Since gas cost is subjected to change the fixed gas cost might not be enough to successfully transfer the ether and the overall function call might fail.

Using `call` to transfer ether opens up the possibility of a reentrancy attack since the gas supplied can be used to reenter the function by calling it again inside the receive or fallback function of the receiving contract. Reentrancy can be solved by using a reentrancy guard.
 
An example of how to use `call` with [reentrancy guard](https://docs.openzeppelin.com/contracts/4.x/api/security#ReentrancyGuard)
```solidity
function paySomeone(address _addr) public payable nonReentrant {
    (bool success, ) = _addr.call{value: msg.value}("");
    // do something
}
``` 

## call vs delegatecall
`delegatecall` is used to call a function of contract B from contract A with contract A’s storage, balance and address supplied to the function. This is done for the purpose of using the function in contract B as library code. Since the function will behave as it was a function of Contract A itself. Check [this](https://solidity-by-example.org/delegatecall/) post for a code example. 

`delegatecall` syntax is exactly the same as `call` syntax except it cannot accept the `value` option but only `gas`.
