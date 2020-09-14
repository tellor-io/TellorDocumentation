## Using Tellor

Tellor's native currency, TRB, is an ERC20 token that also keeps checkpoint balances. Tellor uses a network of staked PoW miners that compete to provide data on-chain and they are rewarded with TRB. Similar to the way Ethereum's miners are incentivized to add transactions to the block through gas, "tips" are used to incentivize TRB miners to add data on-chain. Miners will add the data for the request with the highest tip. The validity of the data can be disputed and settled by a community vote. Once the data is on-chain it can be read by all smart contracts.

Tellor is deployed on mainnet and Rinkeby testnet. You can get TRB through several decentralized exchanges. For Rinkeby testnet tokens contact us at info@tellor.io.

Mainnet Address- [0x0ba45a8b5d5575935b8158a88c631e9f9c95a2e5](https://etherscan.io/address/0x0ba45a8b5d5575935b8158a88c631e9f9c95a2e5)

Rinkeby Address- [0xFe41Cb708CD98C5B20423433309E55b53F79134a](https://rinkeby.etherscan.io/address/0xFe41Cb708CD98C5B20423433309E55b53F79134a)

### Get and Read Data
Using Tellor requires two actions: 1) incentivize miners to add your data on-chain using the "addTip" function and 2) read data using the functions in the [UserSetup](./UserSetup.md).

To view the queue and tip and add tip to a data request go to TellorData at [http://prices.tellorscan.com/](http://prices.tellorscan.com/).

Alternatively, "addTip" can also be called directly from Tellor. These functions can be used to determine how much tip to add:

* Use the "getRequestVars" function to Check where your request sits on the queue and tip associated with it.

```solidity
getRequestVars(uint256 _requestId) external view returns (uint256 index, uint256 tip) 
```

* Use the "getVariablesOnDeck()" function to check the tip associated with the requestId on deck(next to be mined)

```solidity
getVariablesOnDeck() external view returns (uint256 _requestId, uint256 _tip)
```

* Use the "addTip" function to bring the requestId to the top of the queue (next on queue is always the requestId with the highest tip)

```solidity
    addTip(uint _requestId, uint _tip)
```

### Disputes
The validity of the data can be disputed by any TRB holder for a fee. TRB holders vote on the validity of the data and if the miner is found to be malicious their stake goes to the party that initiated the dispute, otherwise the dispute fee goes to the miner. If you believe the data is invalid, initiate a dispute. However, you do not need to wait for the dispute to be voted on to use the data, simply addTip to your required request and the data will be added on-chain soon after(approximately 10 mins).

To initiate a dispute use our dispute center at [http://disputes.tellorscan.com/](http://disputes.tellorscan.com/)

Alternatively, you can call the following functions from Tellor contract to begin a dispute, vote on it and tally the votes:

* beginDispute

```solidity
beginDispute(uint256 _requestId, uint256 _timestamp, uint256 _minerIndex)
``` 

* vote

After a dispute is initiated. Use this function to vote on the validity of the data. 

```solidity
vote(uint256 _disputeId, bool _supportsDispute)
```

* tallyVotes

Once the dispute period has passed anyone can run the tallyVotes function. This function tallies the votes and pays out either the miner or the party that initiated the dispute. 

```solidity
tallyVotes(uint256 _disputeId)
```

