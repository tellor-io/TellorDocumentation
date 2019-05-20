

## User functions <a name="user-fx"> </a>  

## requestData
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

## addTip

* <b>addTip</b> function allows the user to increase the payout for a specific request

```solidity
    function addTip(uint _requestId, uint _tip) external {
        tellor.addTip(_requestId,_tip);
    }
```

where:
  * \_requestId is the ID for the value to be mined
  * \_tip is the amount the requester is willing to pay to be get on queue

## retrieveData
To read data, users will need to call these two functions: 

* <b>retreiveData</b> function allows the user to read the data for the given API and timestamp
```javascript
oracle.retrieveData(uint _requestId, uint _timestamp)
```
where:

  * \_requestId -- is the request ID
  * \_timestamp -- is the unix timestamp to retrieve a value from

## getLastQuery

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
