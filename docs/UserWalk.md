## Using Tellor

Tellor's native currency, TRB, is an ERC20 token that also keeps checkpoint balances. Tellor uses a network of staked PoW miners that compete to provide data on-chain and they are rewarded with TRB. Similar to the way Ethereum's miners are incentivized to add transactions to the block through gas, "tips" are used to incentivize TRB miners to add data on-chain. Miners will add the data for the request with the highest tip. Once the data is on-chain it can be read by all smart contracts.

Tellor is deployed in mainnet and Rinkeby testnet. You can get TRB through several decentralized exchanges. For Rinkeby testnet tokens contact us at info@tellor.io.

As a user you will need two actions: 1) incentivize miners to add your data on-chain using "addTip" function 2) read data using the read function in the [UserSetup](./UserSetup.md).

To view the queue and tip and add tip to a data request go to TellorData at [http://prices.tellorscan.com/](http://prices.tellorscan.com/).

Alternatively, addTip can also be called directly from Tellor. 
Mainnet Address- [0x0ba45a8b5d5575935b8158a88c631e9f9c95a2e5](https://etherscan.io/address/0x0ba45a8b5d5575935b8158a88c631e9f9c95a2e5)

Rinkeby Address- [0xFe41Cb708CD98C5B20423433309E55b53F79134a](https://rinkeby.etherscan.io/address/0xFe41Cb708CD98C5B20423433309E55b53F79134a)

These functions can be used to determine how much tip to add:

* "getRequestVars" to Check where your request sits on the queue and tip associated with it by calling the getRequestVars function.

```solidity
getRequestVars(uint256 _requestId)getRequestVars(uint256 _requestId) external view returns (uint256 index, uint256 tip) 
```

* "getVariablesOnDeck()" Check the tip associated with the requestId on deck(next to be mined)

```solidity
getVariablesOnDeck() external view returns (uint256 requestId, uint256 tip)
```

Use the "addTip" function to bring the requestId to the top of the queue (next on queue is alays the requestId with the highest tip)

```solidity
    addTip(uint _requestId, uint _tip)
```


