# <span style="color:#06D88C"> Tellor </span>


## User functions <a name="user-fx"> </a>  

To request data, users will need Tributes to call this function:
* <b>requestData</b> function allows the user to specify the API, timestamp and tip (this can be thought of as “gas”, the higher the tip/payout the higher the probability it will get mined next) for the value they are requesting.  If multiple parties are requesting the same data at the same time, their tips are combined to further incentivize miners at that time period and/or API. 

```javascript
oracle.requestData(string calldata _c_sapi,string calldata _c_symbol,uint _requestId,uint _granularity, uint _tip)
```
where:
   * \_c_sapi string API being requested be mined
   * \_c_symbol is the short string symbol for the api request
   * \_requestId being requested be mined if it exist otherwise use zero(0)
   * \_granularity is the number of decimals miners should include on the submitted value
   * \_tip amount the requester is willing to pay to be get on queue. Miners mine the onDeckQueryHash, or the api with the highest payout pool


To read data, users will need to call these two functions: 
* <b>retreiveData</b> function allows the user to read the data for the given API and timestamp
```javascript
oracle.retrieveData(uint _requestId, uint _timestamp)
```
where:
  * \_requestId -- is the request ID
  * \_timestamp -- is the unix timestamp to retrieve a value from

* <b>getLastQuery</b> function allows the user to read data from the latest API and timestamp mined. 
```javascript
oracle.getLastNewValue()
```

This is an example of a function that would need to be added to a contract so that it can read data from an oracle contact if the contract holds Tributes:
```Solidity
contract Oracle is usingTellor {
             ...
  function getLastNewValue() public returns(uint,bool) {
    (value,ifRetrieve)  = getLastNewValue();
                           return (value, ifRetreive);
             ...
  }
```
## Miner functions <a name="miner-fx"> </a>  
Miners engage in a POW competition to find a nonce which satisfies the requirement of the challenge.  The first five miners who solve the PoW puzzle provide the nonce, requestId, and value and receive native tokens in exchange for their work.  The oracle data submissions are stored in contract memory as an array - which is subsequently operated upon to derive the median value and the miner payout. 

Miners need to extract the current challenge, requestId and difficulty by calling the getCurrentVariables function before they can begin solving the PoW.

```javascript
oracle.getCurrentVariables()
```

The getCurrentVariables solidity function returns all the necessary variables. 
```solidity
    function getCurrentVariables(TellorStorageStruct storage self) internal view returns(bytes32, uint, uint,string memory,uint,uint){    
        return (self.currentChallenge,self.uintVars[keccak256("currentRequestId")],self.uintVars[keccak256("difficulty")],self.requestDetails[self.uintVars[keccak256("currentRequestId")]].queryString,self.requestDetails[self.uintVars[keccak256("currentRequestId")]].apiUintVars[keccak256("granularity")],self.requestDetails[self.uintVars[keccak256("currentRequestId")]].apiUintVars[keccak256("totalTip")]);
    }
```

Miners can use the submitMiningSolution function to submit the PoW, requestId, and off-chain value. Production and test python miners are available under the miner subdirectory [here](./miner/).  The PoW challenge is different than the regular PoW challenge used in Bitcoin. 

```javascript
oracle.submitMiningSolution(string nonce, uint _requestId, uint value)
```
where 
  * nonce -- is the solution string submitted by miner
  * \_requestId -- is the requestId for the request on queue
  * value -- is the value of api query