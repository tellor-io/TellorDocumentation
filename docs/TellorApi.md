# Tellor REST API
The [Tellor REST API](https://github.com/tellor-io/tellorREST) repository allows you to create a local API for Tellor information. You can also access Tellor API using the endpoint at [api.tellorscan.com](http://api.tellorscan.com/info)

# REST API Documentation
This section documents the REST API endpoints.

## GET /info
Get general statistics about the Tellor network.
### Request
* None
### Response
* **stakerCount:** The number of accounts staking 1000 TRB
* **difficulty:** The current network difficulty
* **currentRequestId:** The current request ID, referencing PSR
* **disputeCount:** The number of disputes that have occured all time		
* **total_supply:** The amount of TRB tokens that have been issued by the Tellor network
* **timeOfLastNewValue:** The unix epoch time that the last new value was added to the Tellor data feed
* **requestCount:** Total number of requests through the system
#### Example
```
{
  "stakerCount": "47",
  "difficulty": "1246804904077941",
  "currentRequestId": "2",
  "disputeCount": "19",
  "total_supply": "1145832115753024439922925",
  "timeOfLastNewValue": "1586008020",
  "requestCount": "51"
}
```

## GET /requestq
Get the request queue.
### Request
* None
### Response
* **requestq:** Integer array of the top 50 requests by payment amount
#### Example
```
{
  "requestq": [0, 0, ..., 0]
}
```

## GET /price/:requestID
Get the price of a specific PSR.
### Request
* **requestID:** The API value to request, references a PSR
#### Example
```
curl http://api.tellorscan.com/price/1
```
### Response
* **didGet:** True if the requestID has a price, false otherwise
* **value:** The value for the price, multiplied by some granularity (e.g. 1000)
* **timestampRetrieved:** The unix epoch time the value was last updated
#### Example
```
{
  "didGet": true,
  "value": "136880",
  "timestampRetrieved": "1585835760"
}
```

## GET /requestinfo/:requestID
Get general information about a PSR.
### Request
* **requestID:** The API value to request, references a PSR
#### Example
```
curl http://api.tellorscan.com/requestinfo/1
```
### Response
* **apiString:** The query string for this request
* **dataSymbol:** Short name for API request
* **queryHash:** Hash of api string and granularity
* **granularity:** The amount of decimals desired on the requested value
* **requestQPosition:** Index in request queue
* **totalTip:** Bonus portion of payout
#### Example
```
{
  "apiString": "PSR",
  "dataSymbol": "PSR",
  "queryHash": "0x91791f339a108cc51810558511858b09e35f6deba14e2c05488a31097fe73879",
  "granularity": "1000",
  "requestQPosition": "0",
  "totalTip": "0"
}
```

## GET /dispute/:disputeID
Get information about a dispute.
### Request
* **disputeID:** The dispute ID to request
#### Example
```
curl http://api.tellorscan.com/dispute/19
```
### Response
* **hash:** Unique hash of dispute: keccak256(miner,requestId,timestamp)
* **executed:** True if the dispute settled
* **isPropFork:** True for fork proposal
* **reportedMiner:** Miner who allegedly submitted the 'bad value'
* **reportingParty:** Miner reporting the 'bad value'-pay disputeFee will get reportedMiner's stake if dispute vote passes
* **proposedForkAddress:** New fork address (if fork proposal)
* **requestId:** Information about the request including:
  * RequestID of disputed value
  * Timestamp of disputed value
  * The value being disputed
  * 7 days from when dispute initialized
  * The number of parties who have voted on the measure
  * The block number for which votes will be calculated from
  * Index in dispute array
  * Fee paid corresponding to dispute

#### Example
```
{
  "hash": "0xb35fa004279a349e13b80e9dbeeb144d6f30463866833c331d27d1b95fa9d326",
  "executed": true,
  "disputeVotePassed": true,
  "isPropFork": false,
  "reportedMiner": "0x103348C47fFc3254aFf761894e7C13cA0C680465",
  "reportingParty": "0xbABca74dB0D4dBCb7EBC89728452CeAC807615A0",
  "proposedForkAddress": "0x0000000000000000000000000000000000000000",
  "requestId": [
    "3",
    "1572377640",
    "187880",
    "1572985087",
    "2",
    "8836057",
    "4",
    "80352476817699999998000",
    "510000000000000000000"
  ],
  "timestamp": "80352476817699999998000"
  }
```


## To test:

Requirements: Node.js version 9.2.0

1. Clone the repository

```git
       git clone https://github.com/tellor-io/tellorREST

```

2. Create a .env file based on the example (update the infura key to reflect your key)

3. Run:

```node
		npm install
		node index
```

4. Now visit these urls from your browswer:

* To get general information:		http://localhost:5000/info
* To get information about the request queue: http://localhost:5000/requestq
* To get information about a requestId (api, granularity, etc..): http://localhost:5000/requestinfo/requestID
    * For example: http://localhost:5000/requestinfo/1
* To get price inforamtion for specified requestId: http://localhost:5000/price/requestID
    * For example: http://localhost:5000/price/1
* To get dispute inforamtion for a specific disputeId:  http://localhost:5000/dispute/:disputeID

## Custom API
Use the following hashes to read data from Tellor's contract.

### Common Hashes:
| Function        | value              | keccak-256                                                         |
|-----------------|--------------------|--------------------------------------------------------------------|
| addressVars     | tellorContract     | 0xd48fd09afdab521f4f69bd2af8177f60fb0709ce0f1b3d5b8a2e233a20453848 |
| addressVars     | _owner             | 0x9dbc393ddc18fd27b1d9b1b129059925688d2f2d5818a5ec3ebb750b7c286ea6 |
| addressVars     | _deity             | 0xc72fb71df90ec89e61e8dea6fee5142880a8a329caaae9ff4931955d88f59990 |
| apiUintVars     | granularity        | 0xba3571a50e0c436953d31396edfb65be5925bcc7fef5a3441ed5d43dbce2548f |
| apiUintVars     | requestQPosition   | 0x1e344bd070f05f1c5b3f0b1266f4f20d837a0a8190a3a2da8b0375eac2ba86ea |
| apiUintVars     | totalTip           | 0x2a9e355a92978430eca9c1aa3a9ba590094bac282594bccf82de16b83046e2c3 |
| disputeUintVars | requestId          | 0x31b40192effc42bcf1e4289fe674c678e673a3052992548fef566d8c33a21b91 |
| disputeUintVars | timestamp          | 0xd056b4f4e783ee91bebc956e3ffe3c71aec2992408313c1db5ee11c1b4fa7c41 |
| disputeUintVars | value              | 0x81afeeaff0ed5cee7d05a21078399c2f56226b0cd5657062500cef4c4e736f85 |
| disputeUintVars | minExecutionDate   | 0x74c9bc34b0b2333f1b565fbee67d940cf7d78b5a980c5f23da43f6729965ed40 |
| disputeUintVars | numberOfVotes      | 0xa0bc13ce85a2091e950a370bced0825e58ab3a3ffeb709ed50d5562cbd82faab |
| disputeUintVars | blockNumber        | 0x6f8f54d1af9b6cb8a219d88672c797f9f3ee97ce5d9369aa897fd0deb5e2dffa |
| disputeUintVars | minerSlot          | 0x8ef61a1efbc527d6428ff88c95fdff5c6e644b979bfe67e03cbf88c8162c5fac |
| disputeUintVars | fee                | 0x833b9f6abf0b529613680afe2a00fa663cc95cbdc47d726d85a044462eabbf02 |
| uintVars        | decimals           | 0x784c4fb1ab068f6039d5780c68dd0fa2f8742cceb3426d19667778ca7f3518a9 |
| uintVars        | disputeFee         | 0x8b75eb45d88e80f0e4ec77d23936268694c0e7ac2e0c9085c5c6bdfcfbc49239 |
| uintVars        | disputeCount       | 0x475da5340e76792184fb177cb85d21980c2530616313aef501564d484eb5ca1e |
| uintVars        | total_supply       | 0xb1557182e4359a1f0c6301278e8f5b35a776ab58d39892581e357578fb287836 |
| uintVars        | stakeAmount        | 0x7be108969d31a3f0b261465c71f2b0ba9301cd914d55d9091c3b36a49d4d41b2 |
| uintVars        | stakerCount        | 0xedddb9344bfe0dadc78c558b8ffca446679cbffc17be64eb83973fce7bea5f34 |
| uintVars        | timeOfLastNewValue | 0x97e6eb29f6a85471f7cc9b57f9e4c3deaf398cfc9798673160d7798baf0b13a4 |
| uintVars        | difficulty         | 0xb12aff7664b16cb99339be399b863feecd64d14817be7e1f042f97e3f358e64e |
| uintVars        | currentTotalTips   | 0xd26d9834adf5a73309c4974bf654850bb699df8505e70d4cfde365c417b19dfc |
| uintVars        | currentRequestId   | 0x7584d7d8701714da9c117f5bf30af73b0b88aca5338a84a21eb28de2fe0d93b8 |
| uintVars        | requestCount       | 0x05de9147d05477c0a5dc675aeea733157f5092f82add148cf39d579cafe3dc98 |
| uintVars        | slotProgress       | 0x6c505cb2db6644f57b42d87bd9407b0f66788b07d0617a2bc1356a0e69e66f9a |
| uintVars        | miningReward       | 0x9f355ccb80c88ef4eea7a6d390e83e1044d5676886223220e9522329f054ef16 |
| uintVars        | timeTarget         | 0xad16221efc80aaf1b7e69bd3ecb61ba5ffa539adf129c3b4ffff769c9b5bbc33 |
