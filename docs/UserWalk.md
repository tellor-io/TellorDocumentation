## Documentation under construction

### Step 1: Ensure the provided data source returns a value
Link to our tool to check this: Coming soon!

### Step 2: Get Funds
Contact us at info@tellor.io.

### Step 3: Request Data

* <b>If a request does NOT exist</b>

    If a request does <b> NOT</b> exist for the data source and granularity(number of decimal places) on the list:
	
    Use the function <b>requestData</b> to submit the data source link and specify the granularity and amount willing to pay to get to the top of the queue. 

```solidity
    requestData(string calldata _c_sapi,string calldata _c_symbol,uint _requestId,uint _granularity, uint _tip)
```


   where:

   * \_c_sapi string API being requested be mined
   * \_c_symbol is the short string symbol for the api request
   * \_requestId being requested be mined if it exist otherwise use zero(0)
   * \_granularity is the number of decimals miners should include on the submitted value
   * \_tip amount the requester is willing to pay to be get on queue. Miners mine the onDeckQueryHash, or the api with the highest payout pool

>

* <b> If a request exists </b>

    If a request exists for the data source and granularity (number of decimal places) on the list use the add tip functions to raise your prefered request to the top of the queue.
	  
    If your contract is funded with Tellor Tributes use the function <b> addTip </b> to add to the payout by specifying the amount to add to the payout (increase the miner tip).

```solidity
    addTip(uint _requestId, uint _tip)
```

where:

  * \_requestId is the ID for the value to be mined
  * \_tip is the amount the requester is willing to pay to be get on queue in Tributes

    If your contract is funded with Ether use the <b> addTipWithEther</b> function to tip the miners. This function automatically buys Tellor Tributes to fund your tip. 

```solidity
    addTipWithEther(uint _requestId, uint _tip)
```



### Step 4: Retreive Data or get last query value

Use the function <b> retreiveData </b> to retreive a value by specifying the request ID and timestamp to retreive.

```javascript
    retrieveData(uint _requestId, uint _timestamp)
```
where:

  * \_requestId -- is the request ID
  * \_timestamp -- is the unix timestamp to retrieve a value from


To retreive the latest query value use <b> getLastQuery </b> to get the last mined value.

```javascript
    getLastNewValue()
```
