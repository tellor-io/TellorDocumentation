## Integrate Tellor

To integrate Tellor as your smart contracts' source of data you need to integrate the user contract, UsingTellor.sol into your smart contract.

UsingTellor.sol allows your contract to easily access the functions needed to request, tip and retreive data by interacting with UserContract.sol. 

## Quick install with npm
To quickly install Tellor for testing into your project run the following command: 

   npm install usingtellor

## User Contracts Descriptions

* <b>UserContract.sol</b> - Will exhange ether for Tributes for the data requests or tips submitted with Ether and submit the transaction to Tellor (since request data and tipping transactions for Tellor are required to be in Tributes). The price for Tributes is set by the UserContract.sol owner, a centralized source. In general, because the user will have the option to buy Tributes from other sources, the Tributes price in this contract should not greatly exceed the current market price.

UserContract address on Rinkeby: [0x19e77D1B96978713fca53d946d4392f4b8F3c5AD](https://rinkeby.etherscan.io/address/0x19e77d1b96978713fca53d946d4392f4b8f3c5ad)

UserContract address on Mainnet: [0xCaC3937932621F62D94aCdE77bBB2a091FD26f58](https://etherscan.io/address/0xcac3937932621f62d94acde77bbb2a091fd26f58)

<details>
  <summary>Click to view UserContract.sol!</summary>

```Solidity
pragma solidity ^0.5.0;

import "../TellorMaster.sol";
import "../Tellor.sol";

/**
* @title UserContract
* This contracts creates for easy integration to the Tellor Tellor System
* This contract holds the Ether and Tributes for interacting with the system
* Note it is centralized (we can set the price of Tellor Tributes)
* Once the tellor system is running, this can be set properly.
* Note deploy through centralized 'Tellor Master contract'
*/
contract UserContract {
    //in Loyas per ETH.  so at 200$ ETH price and 3$ Trib price -- (3/200 * 1e18)
    uint256 public tributePrice;
    address payable public owner;
    address payable public tellorStorageAddress;
    Tellor _tellor;
    TellorMaster _tellorm;

    event OwnershipTransferred(address _previousOwner, address _newOwner);
    event NewPriceSet(uint256 _newPrice);

    /*Constructor*/
    /**
    * @dev the constructor sets the storage address and owner
    * @param _storage is the TellorMaster address ???
    */
    constructor(address payable _storage) public {
        tellorStorageAddress = _storage;
        _tellor = Tellor(tellorStorageAddress); //we should delcall here
        _tellorm = TellorMaster(tellorStorageAddress);
        owner = msg.sender;
    }

    /*Functions*/
    /**
    * @dev Allows the current owner to transfer control of the contract to a newOwner.
    * @param newOwner The address to transfer ownership to.
    */
    function transferOwnership(address payable newOwner) external {
        require(msg.sender == owner, "Sender is not owner");
        emit OwnershipTransferred(owner, newOwner);
        owner = newOwner;
    }

    /**
    * @dev This function allows the owner to withdraw the ETH paid for requests
    */
    function withdrawEther() external {
        require(msg.sender == owner, "Sender is not owner");
        owner.transfer(address(this).balance);
    }

    /**
    * @dev Allows the contract owner(Tellor) to withdraw any Tributes left on this contract
    */
    function withdrawTokens() external {
        require(msg.sender == owner, "Sender is not owner");
        _tellor.transfer(owner, _tellorm.balanceOf(address(this)));
    }

    /**
    * @dev Allows the user to submit a request for data to the oracle using ETH
    * @param c_sapi string API being requested to be mined
    * @param _c_symbol is the short string symbol for the api request
    * @param _granularity is the number of decimals miners should include on the submitted value
    * @param _tip amount the requester is willing to pay to be get on queue. Miners
    * mine the onDeckQueryHash, or the api with the highest payout pool
    */
    function requestDataWithEther(string calldata c_sapi, string calldata _c_symbol, uint256 _granularity, uint256 _tip) external payable {
        require(_tellorm.balanceOf(address(this)) >= _tip, "Balance is lower than tip amount");
        require(msg.value >= (_tip * tributePrice) / 1e18, "Value is too low");
        _tellor.requestData(c_sapi, _c_symbol, _granularity, _tip);
    }

    /**
    * @dev Allows the user to tip miners using ether
    * @param _apiId to tip
    * @param _tip amount
    */
    function addTipWithEther(uint256 _apiId, uint256 _tip) external payable {
        require(_tellorm.balanceOf(address(this)) >= _tip, "Balance is lower than tip amount");
        require(msg.value >= (_tip * tributePrice) / 1e18, "Value is too low");
        _tellor.addTip(_apiId, _tip);
    }

    /**
    * @dev Allows the owner to set the Tribute token price.
    * @param _price to set for Tellor Tribute token
    */
    function setPrice(uint256 _price) public {
        require(msg.sender == owner, "Sender is not owner");
        tributePrice = _price;
        emit NewPriceSet(_price);
    }

    /**
    * @dev Allows the user to get the latest value for the requestId specified
    * @param _requestId is the requestId to look up the value for
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp
    */
    function getCurrentValue(uint256 _requestId) public view returns (bool ifRetrieve, uint256 value, uint256 _timestampRetrieved) {
        uint256 _count = _tellorm.getNewValueCountbyRequestId(_requestId);
        if (_count > 0) {
            _timestampRetrieved = _tellorm.getTimestampbyRequestIDandIndex(_requestId, _count - 1); //will this work with a zero index? (or insta hit?)
            return (true, _tellorm.retrieveData(_requestId, _timestampRetrieved), _timestampRetrieved);
        }
        return (false, 0, 0);
    }

    /**
    * @dev Allows the user to get the first verified value for the requestId after the specified timestamp
    * @param _requestId is the requestId to look up the value for
    * @param _timestamp after which to search for first verified value
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp, the timestamp after
    * which it searched for the first verified value
    */
    function getFirstVerifiedDataAfter(uint256 _requestId, uint256 _timestamp) public view returns (bool, uint256, uint256 _timestampRetrieved) {
        uint256 _count = _tellorm.getNewValueCountbyRequestId(_requestId);
        if (_count > 0) {
            for (uint256 i = _count; i > 0; i--) {
                if (
                    _tellorm.getTimestampbyRequestIDandIndex(_requestId, i - 1) > _timestamp &&
                    _tellorm.getTimestampbyRequestIDandIndex(_requestId, i - 1) < block.timestamp - 86400
                ) {
                    _timestampRetrieved = _tellorm.getTimestampbyRequestIDandIndex(_requestId, i - 1); //will this work with a zero index? (or insta hit?)
                }
            }
            if (_timestampRetrieved > 0) {
                return (true, _tellorm.retrieveData(_requestId, _timestampRetrieved), _timestampRetrieved);
            }
        }
        return (false, 0, 0);
    }

    /**
    * @dev Allows the user to get the first value for the requestId after the specified timestamp
    * @param _requestId is the requestId to look up the value for
    * @param _timestamp after which to search for first verified value
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp
    */
    function getAnyDataAfter(uint256 _requestId, uint256 _timestamp)
        public
        view
        returns (bool _ifRetrieve, uint256 _value, uint256 _timestampRetrieved)
    {
        uint256 _count = _tellorm.getNewValueCountbyRequestId(_requestId);
        if (_count > 0) {
            for (uint256 i = _count; i > 0; i--) {
                if (_tellorm.getTimestampbyRequestIDandIndex(_requestId, i - 1) >= _timestamp) {
                    _timestampRetrieved = _tellorm.getTimestampbyRequestIDandIndex(_requestId, i - 1); //will this work with a zero index? (or insta hit?)
                }
            }
            if (_timestampRetrieved > 0) {
                return (true, _tellorm.retrieveData(_requestId, _timestampRetrieved), _timestampRetrieved);
            }
        }
        return (false, 0, 0);
    }

}

```
</details>
<br>

* <b>UsingTellor.sol </b> - Facilitates the integration of the Tellor system into smart contracts. It can exchange Tributes for Ether through the UserContract.sol before submitting the transaction to Tellor. The user has the option to get Tributes from a DEX or to buy Tributes directly through this contract from the "UserContract.sol".

<details>
  <summary>Click to view UsingTellor.sol!</summary>

```solidity
pragma solidity ^0.5.0;

import "../Tellor.sol";
import "../TellorMaster.sol";
import "./UserContract.sol";
/**
* @title UsingTellor
* This contracts creates for easy integration to the Tellor Tellor System
*/
contract UsingTellor {
    UserContract tellorUserContract;
    address payable public owner;

    event OwnershipTransferred(address _previousOwner, address _newOwner);

    /*Constructor*/
    /**
    * @dev This function sents the owner and userContract address
    * @param _userContract is the UserContract.sol address
    */
    constructor(address _userContract) public {
        tellorUserContract = UserContract(_userContract);
        owner = msg.sender;
    }

    /*Functions*/
    /**
    * @dev Allows the user to get the latest value for the requestId specified
    * @param _requestId is the requestId to look up the value for
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp
    */
    function getCurrentValue(uint256 _requestId) public view returns (bool ifRetrieve, uint256 value, uint256 _timestampRetrieved) {
        return tellorUserContract.getCurrentValue(_requestId);
    }

    //How can we make this one more efficient?
    /**
    * @dev Allows the user to get the first verified value for the requestId after the specified timestamp
    * @param _requestId is the requestId to look up the value for
    * @param _timestamp after which to search for first verified value
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp, the timestamp after
    * which it searched for the first verified value
    */
    function getFirstVerifiedDataAfter(uint256 _requestId, uint256 _timestamp) public view returns (bool, uint256, uint256 _timestampRetrieved) {
        return tellorUserContract.getFirstVerifiedDataAfter(_requestId, _timestamp);
    }

    /**
    * @dev Allows the user to get the first value for the requestId after the specified timestamp
    * @param _requestId is the requestId to look up the value for
    * @param _timestamp after which to search for first verified value
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp
    */
    function getAnyDataAfter(uint256 _requestId, uint256 _timestamp)
        public
        view
        returns (bool _ifRetrieve, uint256 _value, uint256 _timestampRetrieved)
    {
        return tellorUserContract.getAnyDataAfter(_requestId, _timestamp);
    }

    /**
    * @dev Allows the user to submit a request for data to the oracle using Tributes
    * Allowing this prevents us from increasing spread too high (since if we set the price too hight
    * users will just go to an exchange and get tokens from there)
    * @param _request string API being requested to be mined
    * @param _symbol is the short string symbol for the api request
    * @param _granularity is the number of decimals miners should include on the submitted value
    * @param _tip amount the requester is willing to pay to be get on queue. Miners
    * mine the onDeckQueryHash, or the api with the highest payout pool
    */
    function requestData(string calldata _request, string calldata _symbol, uint256 _granularity, uint256 _tip) external {
        Tellor _tellor = Tellor(tellorUserContract.tellorStorageAddress());
        if (_tip > 0) {
            require(_tellor.transferFrom(msg.sender, address(this), _tip), "Transfer failed");
        }
        _tellor.requestData(_request, _symbol, _granularity, _tip);
    }

    /**
    * @dev Allows the user to submit a request for data to the oracle using ETH
    * @param _request string API being requested to be mined
    * @param _symbol is the short string symbol for the api request
    * @param _granularity is the number of decimals miners should include on the submitted value
    * @param _tip amount the requester is willing to pay to be get on queue. Miners
    * mine the onDeckQueryHash, or the api with the highest payout pool
    */
    function requestDataWithEther(string calldata _request, string calldata _symbol, uint256 _granularity, uint256 _tip) external payable {
        tellorUserContract.requestDataWithEther.value(msg.value)(_request, _symbol, _granularity, _tip);
    }

    /**
    * @dev Allows the user to tip miners for the specified request using Tributes
    * @param _requestId to tip
    * @param _tip amount
    */
    function addTip(uint256 _requestId, uint256 _tip) public {
        Tellor _tellor = Tellor(tellorUserContract.tellorStorageAddress());
        require(_tellor.transferFrom(msg.sender, address(this), _tip), "Transfer failed");
        _tellor.addTip(_requestId, _tip);
    }

    /**
    * @dev Allows user to add tip with Ether by sending the ETH to the userContract and exchanging it for Tributes
    * at the price specified by the userContract owner.
    * @param _requestId to tip
    * @param _tip amount
    */
    function addTipWithEther(uint256 _requestId, uint256 _tip) public payable {
        UserContract(tellorUserContract).addTipWithEther.value(msg.value)(_requestId, _tip);
    }

    /**
    * @dev allows owner to set the user contract address
    * @param _userContract address
    */
    function setUserContract(address _userContract) public {
        require(msg.sender == owner, "Sender is not owner"); //who should this be?
        tellorUserContract = UserContract(_userContract);
    }

    /**
    * @dev allows owner to transfer ownership
    * @param _newOwner address
    */
    function transferOwnership(address payable _newOwner) external {
        require(msg.sender == owner, "Sender is not owner");
        emit OwnershipTransferred(owner, _newOwner);
        owner = _newOwner;
    }
}


```
</details>
<br>

## Instructions

Import UsingTellor.sol into your smart contract and ensure your contract inherits from it by adding "is UsingTellor".

Through your contract's constructor function pass through the UserContract.sol address to the UsingTellor.sol contract similar to the contstructor function shown below. 

```solidity
pragma solidity ^0.5.0;

import './UsingTellor.sol';
import '../TellorMaster.sol';
import '../Tellor.sol';

contract YourContract is UsingTellor{
 ...
 	constructor(address _userContract) UsingTellor(_userContract) public{

	}	

	function getFirstUndisputedValueAfter(uint _timestamp) public view returns(bool,uint, uint _timestampRetrieved){
		uint _count = timestamps.length;
		if(_count > 0){
				for(uint i = _count;i > 0;i--){
					if(timestamps[i-1] >= _timestamp && disputedValues[timestamps[i-1]] == false){
						_timestampRetrieved = timestamps[i-1];
					}
				}
				if(_timestampRetrieved > 0){
					return(true,getMyValuesByTimestamp(_timestampRetrieved),_timestampRetrieved);
				}
        }
        return(false,0,0);
	}	
 ...
}
```

Now you can access all the functions within UsingTellor to requestData, addTip, retreiveData, getLastQuery by using Tributes or Ether.


## Example of using Tellor as a dispute mechanism

To implement Tellor as a dispute mechanism, you need a contract that will read data off of the Tellor Oracle System.  In this example, we use two contracts along with the address for the UserContract and Using_Tellor.

* TestContract.sol -- Is a test contract where we create a test contract and settle it using the Optimistic.sol oracle contract. However, for production, you would replace these two functions with real (your) functions that need to read data off of the Tellor Oracle System. 

<details>
  <summary>Click to view Reader.sol!</summary>

```solidity
pragma solidity ^0.5.0;

import "./Optimistic.sol";
/**
* @title Reader
* This contracts is a pretend contract using Tellor that compares two time values
*/
contract TestContract is Optimistic {
    uint256 public startDateTime;
    uint256 public endDateTime;
    uint256 public startValue;
    uint256 public endValue;
    bool public longWins;
    bool public contractEnded;
    event ContractSettled(uint256 _svalue, uint256 _evalue);
    /**
    * @dev This constructor function is used to pass variables to the optimistic contract's constructor
    * and the function is blank
    * @param _userContract address for UserContract
    * @param _disputeFeeRequired the fee to dispute the optimistic price(price sumbitted by known trusted party)
    * @param _disputePeriod is the time frame a value can be disputed after being imputed
    * @param _requestIds are the requests Id's on the Tellor System corresponding to the data types used on this contract.
    * It is recommended to use several requestId's that pull from several API's. If requestsId's don't exist in the Tellor
    * System be sure to create some.
    * @param _granularity is the amount of decimals desired on the requested value
    */
    constructor(address _userContract, uint256 _disputeFeeRequired, uint256 _disputePeriod, uint256[] memory _requestIds, uint256 _granularity)
        public
        Optimistic(_userContract, _disputeFeeRequired, _disputePeriod, _requestIds, _granularity)
    {}

    /**
    * @dev creates a start(now) and end time(now + duration specified) for testing a contract start and end period
    * @param _duration in seconds
    */
    function testContract(uint256 _duration) external {
        startDateTime = now - (now % granularity);
        endDateTime = now - (now % granularity) + _duration;
    }

    /**
    * @dev testing function that settles the contract by getting the first undisputed value after the startDateTime
    * and the first undisputed value after the end time of the contract and settleling(payin off) it.
    */
    function settleContracts() external {
        bool _didGet;
        uint256 _time;
        (_didGet, startValue, _time) = getFirstUndisputedValueAfter(startDateTime);
        if (_didGet) {
            (_didGet, endValue, _time) = getFirstUndisputedValueAfter(endDateTime);
            if (_didGet) {
                if (endValue > startValue) {
                    longWins = true;
                }
                contractEnded = true;
                emit ContractSettled(startValue, endValue);
            }
        }
    }
}

```
</details>
<br>

* Optimistic.sol -- is an optimistic oracle that relies on a centralized price feed, allows disputes on the price feed and resolves the dispute using the Tellor Oracle System by retreiving data from and requesting it as needed. The disputer is able to make a data request using either Tellor Tributes or Ether. Optimistic.sol is UsingTellor.sol and passes the UserContract address through the constructor.

<details>
  <summary>Click to view Optimistic.sol!</summary>

```solidity
pragma solidity ^0.5.0;

import "./UsingTellor.sol";
import "../TellorMaster.sol";
import "../Tellor.sol";
/**
* @title UsingTellor
* This contracts creates for easy integration to the Tellor Tellor System
*/
contract Optimistic is UsingTellor {
    //Can we rework some of these mappings into a struct?
    mapping(uint256 => bool) public isValue; //mapping for timestamp to bool where it's true if the value as been set
    mapping(uint256 => uint256) valuesByTimestamp; //mapping of timestamp to value
    mapping(uint256 => bool) public disputedValues; //maping of timestamp to bool where it's true if the value has been disputed
    mapping(uint256 => uint256[]) public requestIdsIncluded; //mapping of timestamp to requestsId's to include

    // struct infoForTimestamp {
    // 	bool isValue;
    // 	uint valuesByTimestamp;
    // 	bool disputedValues;
    // 	uint[] requestIdsIncluded;
    // }
    // mapping(uint => infoForTimestamp) public infoForTimestamps;//mapping timestampt to InfoForTimestamp struct

    uint256[] timestamps; //timestamps with values
    uint256[] requestIds;
    uint256[] disputedValuesArray;
    uint256 public granularity;
    uint256 public disputeFee; //In Tributes
    uint256 public disputePeriod;

    event NewValueSet(uint256 indexed _timestamp, uint256 _value);
    event ValueDisputed(address _disputer, uint256 _timestamp, uint256 _value);
    event TellorValuePlaced(uint256 _timestamp, uint256 _value);

    /*Constructor*/
    /**
    * @dev This constructor function is used to pass variables to the UserContract's constructor and set several variables
    * the variables for the Optimistic.sol constructor come for the Reader.Constructor function.
    * @param _userContract address for UserContract
    * @param _disputeFeeRequired the fee to dispute the optimistic price(price sumbitted by known trusted party)
    * @param _disputePeriod is the time frame a value can be disputed after being imputed
    * @param _requestIds are the requests Id's on the Tellor System corresponding to the data types used on this contract.
    * It is recommended to use several requestId's that pull from several API's. If requestsId's don't exist in the Tellor
    * System be sure to create some.
    * @param _granularity is the amount of decimals desired on the requested value
    */
    constructor(address _userContract, uint256 _disputeFeeRequired, uint256 _disputePeriod, uint256[] memory _requestIds, uint256 _granularity)
        public
        UsingTellor(_userContract)
    {
        disputeFee = _disputeFeeRequired;
        disputePeriod = _disputePeriod;
        granularity = _granularity;
        requestIds = _requestIds;
    }

    /*Functions*/
    /**
    * @dev allows contract owner, a centralized party to enter value
    * @param _timestamp is the timestamp for the value
    * @param _value is the value for the timestamp specified
    */
    function setValue(uint256 _timestamp, uint256 _value) external {
        //Only allows owner to set value
        require(msg.sender == owner, "Sender is not owner");
        //Checks that no value has already been set for the timestamp
        require(getIsValue(_timestamp) == false, "Timestamp is already set");
        //sets timestamp
        valuesByTimestamp[_timestamp] = _value;
        //sets isValue to true once value is set
        isValue[_timestamp] = true;
        //adds timestamp to the timestamps array
        timestamps.push(_timestamp);
        //lets the network know a new timestamp and value have been added
        emit NewValueSet(_timestamp, _value);

    }

    /**
    * @dev allows user to initiate dispute on the value of the specified timestamp
    * @param _timestamp is the timestamp for the value to be disputed
    */
    function disputeOptimisticValue(uint256 _timestamp) external payable {
        require(msg.value >= disputeFee, "Value is below dispute fee");
        //require that isValue for the timestamp being disputed to exist/be true
        require(isValue[_timestamp], "Value for the timestamp being disputed doesn't exist");
        // assert disputePeriod is still open
        require(now - (now % granularity) <= _timestamp + disputePeriod, "Dispute period is closed");
        //set the disputValues for the disputed timestamp to true
        disputedValues[_timestamp] = true;
        //add the disputed timestamp to the diputedValues array
        disputedValuesArray.push(_timestamp);
        emit ValueDisputed(msg.sender, _timestamp, valuesByTimestamp[_timestamp]);
    }

    /**
    * @dev This function gets the Tellor requestIds values for the disputed timestamp. It averages the values on the
    * requestsIds and replaces the value set by the contract owner, centralized party.
    * @param _timestamp to get Tellor data from
    * @return uint of new value and true if it was able to get Tellor data
    */
    function getTellorValues(uint256 _timestamp) public returns (uint256 _value, bool _didGet) {
        //We need to get the tellor value within the granularity.  If no Tellor value is available...what then?  Simply put no Value?
        //No basically, the dispute period for anyValue is within the granularity
        TellorMaster _tellor = TellorMaster(tellorUserContract.tellorStorageAddress());
        Tellor _tellorCore = Tellor(tellorUserContract.tellorStorageAddress());
        uint256 _retrievedTimestamp;
        uint256 _initialBalance = _tellor.balanceOf(address(this)); //Checks the balance of Tellor Tributes on this contract
        //Loops through all the Tellor requestsId's initially(in the constructor) associated with this contract data
        for (uint256 i = 1; i <= requestIds.length; i++) {
            //Get all values for that requestIds' timestamp
            //Check if any is after your given timestamp
            //If yes, return that value. If no, then request that Id
            (_didGet, _value, _retrievedTimestamp) = getFirstVerifiedDataAfter(i, _timestamp);
            if (_didGet) {
                uint256 _newTime = _retrievedTimestamp - (_retrievedTimestamp % granularity); //why are we using the mod granularity???
                //provides the average of the requests Ids' associated with this price feed
                uint256 _newValue = (_value + valuesByTimestamp[_newTime] * requestIdsIncluded[_newTime].length) /
                    (requestIdsIncluded[_newTime].length + 1);
                //Add the new timestamp and value (we don't replace???)
                valuesByTimestamp[_newTime] = _newValue;
                emit TellorValuePlaced(_newTime, _newValue);
                //records the requests Ids included on the price average where all prices came from Tellor requests Ids
                requestIdsIncluded[_newTime].push(i); //how do we make sure it's not called twice?
                //if the value for the newTime does not exist, then push the value, update the isValue to true
                //otherwise if the newTime is under dsipute then update the dispute status to false
                // ??? should the else be an "and"
                if (isValue[_newTime] == false) {
                    timestamps.push(_newTime);
                    isValue[_newTime] = true;
                    emit NewValueSet(_newTime, _value);
                } else if (disputedValues[_newTime] == true) {
                    disputedValues[_newTime] = false;
                }
                //otherwise request the ID and split the contracts initial tributes balance to equally tip all
                //requests Ids associated with this price feed
            } else if (_tellor.balanceOf(address(this)) > requestIds.length) {
                //Request Id to be mined by adding to it's tip
                _tellorCore.addTip(i, _initialBalance / requestIds.length);
            }
        }
    }

    /**
    * @dev Allows the contract owner(Tellor) to withdraw ETH from this contract
    */
    function withdrawETH() external {
        require(msg.sender == owner, "Sender is not owner");
        address(owner).transfer(address(this).balance);
    }

    /**
    * @dev Get the first undisputed value after the timestamp specified. This function is used within the getTellorValues
    * but can be used on its own.
    * @param _timestamp to search the first undisputed value there after
    */
    function getFirstUndisputedValueAfter(uint256 _timestamp) public view returns (bool, uint256, uint256 _timestampRetrieved) {
        uint256 _count = timestamps.length;
        if (_count > 0) {
            for (uint256 i = _count; i > 0; i--) {
                if (timestamps[i - 1] >= _timestamp && disputedValues[timestamps[i - 1]] == false) {
                    _timestampRetrieved = timestamps[i - 1];
                }
            }
            if (_timestampRetrieved > 0) {
                return (true, getMyValuesByTimestamp(_timestampRetrieved), _timestampRetrieved);
            }
        }
        return (false, 0, 0);
    }

    /*Getters*/
    /**
    * @dev Getter function for the value based on the timestamp specified
    * @param _timestamp to retreive value from
    */
    function getMyValuesByTimestamp(uint256 _timestamp) public view returns (uint256 value) {
        return valuesByTimestamp[_timestamp];
    }

    /**
    * @dev Getter function for the number of RequestIds associated with a timestamp, based on the timestamp specified
    * @param _timestamp to retreive number of requestIds
    * @return uint count of number of values for the spedified timestamp
    */
    function getNumberOfValuesPerTimestamp(uint256 _timestamp) external view returns (uint256) {
        return requestIdsIncluded[_timestamp].length;
    }

    /**
    * @dev Checks to if a value exists for the specifived timestamp
    * @param _timestamp to verify
    * @return ture if it exists
    */
    function getIsValue(uint256 _timestamp) public view returns (bool) {
        return isValue[_timestamp];
    }

    /**
    * @dev Getter function for latest value available
    * @return latest value available
    */
    function getCurrentValue() external view returns (uint256) {
        require(timestamps.length > 0, "Timestamps' length is 0");
        return getMyValuesByTimestamp(timestamps[timestamps.length - 1]);
    }

    /**
    * @dev Getter function for the timestamps available
    * @return uint array of timestamps available
    */
    function getTimestamps() external view returns (uint256[] memory) {
        return timestamps;
    }

    /**
    * @dev Getter function for the requests Ids' from Tellor associated with this price feed
    * @return uint array of requests Ids'
    */
    function getRequestIds() external view returns (uint256[] memory) {
        return requestIds;
    }

    /**
    * @dev Getter function for the requests Ids' from Tellor associated with this price feed
    * at the specified timestamp. This only gets populated after a dispute is initiated and the
    * function getTellorValues is ran.
    * @param _timestamp to retreive the requestIds
    * @return uint array of requests Ids' included in the calcluation of the value
    */
    function getRequestIdsIncluded(uint256 _timestamp) external view returns (uint256[] memory) {
        return requestIdsIncluded[_timestamp];
    }

    /**
    * @dev Getter function for the number of disputed values
    * @return uint count of number of values for the spedified timestamp
    */
    function getNumberOfDisputedValues() external view returns (uint256) {
        return disputedValuesArray.length;
    }

    /**
    * @dev Getter function for all disputed values
    * @return the array with all values under dispute
    */
    function getDisputedValues() external view returns (uint256[] memory) {
        return disputedValuesArray;
    }

    /**
    * @dev This checks if the value for the specified timestamp is under dispute
    * @param _timestamp to check if it is under dispute
    * @return true if it is under dispute
    */
    function isDisputed(uint256 _timestamp) external view returns (bool) {
        return disputedValues[_timestamp];
    }

    /**
    * @dev Getter function for the dispute value by index
    * @return the value
    */
    function getDisputedValueByIndex(uint256 _index) external view returns (uint256) {
        return disputedValuesArray[_index];
    }

}


```
</details>
<br>

<b>Instructions:</b> 

###For using Tellor directly on your contracts:

* Ensure your contract inherits from Using Tellor ("is UsingTellor")
* Deploy your contract and specify the UserContact.sol address in the constructor.



###For testing the setup for the contracts described above:
* Deploy the UserContract.sol.
* Using the UserContract.sol address on the constructor for the TestContract.sol, deploy TestContract.sol.

The TestContract.sol contract inherits from Optimisic and UsingTellor.sol. On the Constructor, specifiy the UserContract Address, disputeFeeRequired, disputePeriod, all requestIds from Tellor that correspond to the price feed needed(the function averages all of these to determine the "correct" price), and granularity(the number of decimals to be included on the price value). 




```solidity
contract Reader is Optimistic{

constructor(address _userContract, uint _disputeFeeRequired, uint _disputePeriod, uint[] memory _requestIds, uint _granularity) Optimistic(_userContract,_disputeFeeRequired,_disputePeriod, _requestIds,_granularity) public {}

```

