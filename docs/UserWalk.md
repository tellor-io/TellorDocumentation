## Using Tellor

Tellor's native currency, TRB, is an ERC20 token that also keeps checkpoint balances. Tellor uses a network of staked PoW miners that compete to provide data on-chain and they are rewarded with TRB. Similar to the way Ethereum's miners are incentivized to add transactions to the block through gas, tips are used to incentivize TRB miners to add data on-chain. Once the data is on-chain it can be read by all smart contracts.

Tellor is deployed in mainnet and Rinkeby testnet. You can get TRB through several decentralized exchanges. For Rinkeby testnet tokens contact us at info@tellor.io.

As a user you will need two actions: 1) incentivize miners to add your data on-chain using "addTip" function 2) read data using the read function in the [UserSetup](./UserSetup.md).

To view the queue and tip and add tip to a data request go to TellorData at [http://prices.tellorscan.com/](http://prices.tellorscan.com/).

Alternatively, addTip can also be called directly from a smart contract. To figure out how much tip to add to ensure y your request is next on queue, these functions can be ran:

1) Check where your request sits on the queue and tip associated with it by calling the getRequestVars function.

```solidity
getRequestVars(uint256 _requestId)getRequestVars(uint256 _requestId) external view returns (uint256 index, uint256 tip) 
```

3) Check the tip associated with the requestId on deck(next to be mined)
```solidity
getVariablesOnDeck() external view returns (uint256 requestId, uint256 tip)
```

3) AddTip in an amount to exceed the highest paid tip (the difference between the highest tip and the tip associated with your request)

```solidity
    addTip(uint _requestId, uint _tip)
```

where:

  * \_requestId is the ID for the value to be mined
  * \_tip is the amount the requester is willing to pay to be get on queue in Tributes

