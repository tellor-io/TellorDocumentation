


## Step 1: depositStake()

You need 1000 tributes to be able to run this function. It will automatically stake the msg.sender. 

```javascript
depositStake() 
```

## Step 2: Download miner
Link to download miner: coming soon!

The miner automatically acomplishes this: 

<b>Step 2.1: getCurrentVariables<b>

Miners need to extract the current challenge, requestId and difficulty by calling the getCurrentVariables function before they can begin solving the PoW.

```javascript
getCurrentVariables()
```
The getCurrentVariables solidity function returns all the necessary variables:

* Current Challenge
* Current Request ID
* Difficulty
* API to query
* Granularity
* TotalTip

<b>Step 2.2: Solve POW and get data requested</b>

<b>Step 2.3: submitMiningSolution</b>

```javascript
submitMiningSolution(string nonce, uint _requestId, uint value)
```

where 

  * nonce -- is the solution string submitted by miner
  * \_requestId -- is the requestId for the request on queue
  * value -- is the value of api query