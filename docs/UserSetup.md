## Integrate Tellor

To integrate Tellor as your smart contracts' source of data you need to integrate the user contract, UsingTellor.sol into your smart contract.

UsingTellor.sol allows your contract to easily access the functions needed to request, tip and retreive data. 

## User Contracts Descriptions

* <b>UsingTellor.sol </b> - Facilitates the integration of the Tellor system into smart contracts. It can hold Tributes or Ether. The user has the option to get Tributes from a DEX or to buy Tributes directly through this contract by buying tributes from the "UserContract.sol".

* <b>UserContract.sol</b> - Will exhange ether for Tributes for the data requests or tips submitted with Ether and submit the transaction to Tellor (since request data and tipping transactions for tellor are required to be in Tributes). The price for Tributes is set by the UserContract.sol owner, a centralized source.

## Instructions

Import UsingTellor.sol into your smart contract and ensure your contract inherits from it by adding "is UsingTellor".

```solidity
pragma solidity ^0.5.0;

import './UsingTellor.sol';


contract YourContract is UsingTellor{
 ...
}
```

Now you can access all the functions within UsingTellor to requestData, addTip, retreiveData, getLastQuery by using Tributes or Ether.