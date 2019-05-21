
Miners are an integral part of Tellor and it is in the best interest of the systme to aling incentives to protect miners and users.

## Download Miner
Link to miner download: coming soon!


Miners are incentivized to provide accurate values through 3 processes:

* [Mining rewards are based upon submission of median value](#uncles)
* [Every miner required to stake 1000 tokens](#staking)
* [Every accepted value can be challenged and put to vote by all Tellor Tribute holders](distputes)

The next three subsections provide further details and goals of these.

## Rewards

<b>Mining rewards are based upon submission of median value</b> <a name="uncles"> </a>

Uncle rewards can be used to reduce the chance of a miner gaining 51% of hashing power a smart contract pays miners to mine uncles. The Tellor Oracle utilizes uncles by rewarding  five miners per data value instead of just one. 

## Staking

<b> Every miner required to stake 1000 tokens </b><a name="staking"> </a>

Miners have to stake 1000 Tributes to be able to mine. Proof-of-stake allows for economic penalties to miners submitting incorrect values. Parties pay to report a miner providing false data, which leads to the report going to a vote (all Tribute holders can vote, the duration of the vote dictates how long the “bad” miner is frozen from mining).  If found guilty, the malicious miner’s stake goes to the reporter; otherwise the fee paid by the reporter is given to the wrongly accused miner. 

## Disputes

<b> Every accepted value can be challenged and put to vote by any Tellor token holder</b> <a name="disputes"> </a>

Blockchains are secured via multiple avenues.  The first is in the random selection process provided by PoW.  The second is that even if the first security measure fails, the blockchain can create forks and different chains until the honest miners win out.  Since our oracle system does not have the ability to fork easily, Tellor implements a finality period of 144 blocks(1 day) after original data submissions.  This allows for parties to challenge data submissions and multiplies the cost to break the network by the number implicit successful confirmations needed (144 blocks without a challenge).



## Mining <a name="mining-process"> </a>

Miners need the following information from the Tellor oracle contract:

* Current Challenge
* Current Request ID
* Difficulty
* API to query
* Granularity
* TotalTip

Miners obtain this information via the <b>getCurrentVariables</b> function.  

```solidity
    function getCurrentVariables(TellorStorageStruct storage self) internal view returns(bytes32, uint, uint,string memory,uint,uint){    
        return (self.currentChallenge,self.uintVars[keccak256("currentRequestId")],self.uintVars[keccak256("difficulty")],self.requestDetails[self.uintVars[keccak256("currentRequestId")]].queryString,self.requestDetails[self.uintVars[keccak256("currentRequestId")]].apiUintVars[keccak256("granularity")],self.requestDetails[self.uintVars[keccak256("currentRequestId")]].apiUintVars[keccak256("totalTip")]);
    }
```

#### The Algorithm
Tello's algorithm is different than that of Bitcoin or Ethereum. 

The PoW, is basically guessing a nonce that produces a hash with a certain number of leading zeros using the randomly selected hash function. When mining became highly competitive for Bitcoin, specialized systems called "application-specific integrated circuit", ASICS, were developed. ASICS are "an integrated circuit customized for a particular use", and in the case of Bitcoin, these are designed to solve the PoW faster and more efficiently that CPU's and GPU's ([Learn more about ASICS](https://en.bitcoin.it/wiki/ASIC)). Currently, mining Bitcoin has become so competitive that most of it is mined via mining pools ([Read more about mining pools](https://en.wikipedia.org/wiki/Mining_pool)). Mining Bitcoin also requires large amounts of electricity and many solo ASICS in areas where electricity is more expensive, have been left without use. 

One of the main challenges for a mineable token or any process that relies on mining is the surplus of solo ASICS currently available since if they are used on a small ecosystem these specialized systems can quickly monopolize it. Tellor’s proof of work challenge is designed to be different than the Bitcoin mining challenge. This setup requires miners to invest a significant amount of time to update the mining algorithm and should disincentivize miners to become part of the ecosystem too early, allowing Tellor to grow and mature before larger players join.

The code to determine a successful mine for a given challenge and difficulty is:
```solidity
function submitMiningSolution(TellorStorageStruct storage self,string memory _nonce, uint _requestId, uint _value) internal{
        ....    
                //Issue the the next challenge
                self.currentChallenge = keccak256(abi.encodePacked(_nonce,self.currentChallenge, blockhash(block.number - 1))); // Save hash for next proof
        ....    
```
The difficulty adjustment is based on the percent difference between the time target and the time it took to solve the previous challenge. For example, if the time target is 10 minutes and the PoW challenge is submitted in 6 minutes, the difficulty will increase by the 40 percent on the next challenge.

An implementation of the miner is described in python in the 'miner' sub directory.  In 'miner.py', the script imports the web3 library, pandas, and various other dependencies to solve the keccak256, sha256, and ripemd160 puzzle.  In submitter.js, the nonce value inputs are submitted to the smart contract on-chain.  To examine the mining script, navigate [here](./miner/).

### Submission 
Miners submitting values must submit the following in order to be a valid submission via the <b>proofOfWork</b> function:
* Successful solution
* request Id
* Value of referenced query

Also, the miner must be staked and the requestId they are submitting must match the request on queue.

All of these requirements are enforced in the proofOfWork function.

```solidity
    /**
    * @dev Proof of work is called by the miner when they submit the solution (proof of work and value)
    * @param _nonce uint submitted by miner
    * @param _requestId the apiId being mined
    * @param _value of api query
    */
    function submitMiningSolution(TellorStorageStruct storage self,string memory _nonce, uint _requestId, uint _value) internal{
        //requre miner is staked
        require(self.stakerDetails[msg.sender].currentStatus == 1);
```





