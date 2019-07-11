## Integrate Tellor

To integrate Tellor as your smart contracts' source of data you need to integrate the user contract, UsingTellor.sol into your smart contract.

UsingTellor.sol allows your contract to easily access the functions needed to request, tip and retreive data. 

## User Contracts Descriptions

* <b>UserContract.sol</b> - Will exhange ether for Tributes for the data requests or tips submitted with Ether and submit the transaction to Tellor (since request data and tipping transactions for Tellor are required to be in Tributes). The price for Tributes is set by the UserContract.sol owner, a centralized source. In general, because user will have the option to buy Tributes from other sources, the Tributes price in this contract should not greatly exceed the current market price.

UserContract address on Rinkeby: Coming soon!

<details>
  <summary>Click to view UserContract.sol!</summary>

```Solidity
pragma solidity ^0.5.0;

import '../TellorMaster.sol';
import '../Tellor.sol';

/**
* @title UserContract
* This contracts creates for easy integration to the Tellor Tellor System
* This contract holds the Ether and Tributes for interacting with the system
* Note it is centralized (we can set the price of Tellor Tributes)
* Once the tellor system is running, this can be set properly.  
* Note deploy through centralized 'Tellor Master contract'
*/
contract UserContract{

	address payable public owner;
	uint public tributePrice;
	address payable public tellorStorageAddress;

	event OwnershipTransferred(address _previousOwner,address _newOwner);
	event NewPriceSet(uint _newPrice);


    /*Constructor*/
    /**
    * @dev the constructor sets the storage address and owner
    * @param _storage is the TellorMaster address ???
    */
    constructor(address payable _storage) public{
    	tellorStorageAddress = _storage;
    	owner = msg.sender;
    }

    /*Functions*/
    /**
    * @dev Allows the current owner to transfer control of the contract to a newOwner.
    * @param newOwner The address to transfer ownership to.
    */
    function transferOwnership(address payable newOwner) external {
            require(msg.sender == owner);
            emit OwnershipTransferred(owner, newOwner);
            owner = newOwner;
    }


    /**
    * @dev This function allows the owner to withdraw the ETH paid for requests
    */
	function withdrawEther() external {
		require(msg.sender == owner);
		owner.transfer(address(this).balance);

	}

	/**
    * @dev Allows the user to submit a request for data to the oracle using ETH
    * @param c_sapi string API being requested to be mined
    * @param _c_symbol is the short string symbol for the api request
    * @param _granularity is the number of decimals miners should include on the submitted value
    * @param _tip amount the requester is willing to pay to be get on queue. Miners
    * mine the onDeckQueryHash, or the api with the highest payout pool
    */
	function requestDataWithEther(string calldata c_sapi, string calldata _c_symbol,uint _granularity, uint _tip) external payable{
		TellorMaster _tellorm = TellorMaster(tellorStorageAddress);
		require(_tellorm.balanceOf(address(this)) >= _tip);
		require(msg.value >= _tip * tributePrice);
		Tellor _tellor = Tellor(tellorStorageAddress); //we should delcall here
		_tellor.requestData(c_sapi,_c_symbol,_granularity,_tip);
	}


    /**
    * @dev Allows the user to tip miners using ether
    * @param _apiId to tip
    * @param _tip amount
    */
	function addTipWithEther(uint _apiId, uint _tip) external payable {
		TellorMaster _tellorm = TellorMaster(tellorStorageAddress);
		require(_tellorm.balanceOf(address(this)) >= _tip);
		require(msg.value >= _tip * tributePrice);
		Tellor _tellor = Tellor(tellorStorageAddress); //we should delcall here
		_tellor.addTip(_apiId,_tip);
	}

    
    /** 
    * @dev Allows the user get the latest tip paid to miners to mine
    * @return uint of latest tip amount paid
    */
    function getLatestTip() public returns(uint latestTip) {
        TellorMaster _tellorm = TellorMaster(tellorStorageAddress);
        latestTip = _tellorm.getUintVar(keccak256("currentTotalTips"));
        return latestTip;
    }


    /**
    * @dev Allows the owner to set the Tribute token price.
    * @param _price to set for Tellor Tribute token
    */
	function setPrice(uint _price) public {
		require(msg.sender == owner);
		tributePrice = _price;
		emit NewPriceSet(_price);
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

import '../Tellor.sol';
import '../TellorMaster.sol';
import './UserContract.sol';
/**
* @title UsingTellor
* This contracts creates for easy integration to the Tellor Tellor System
*/
contract UsingTellor{
	UserContract tellorUserContract;
	address public owner;
	
	event OwnershipTransferred(address _previousOwner,address _newOwner);

    /*Constructor*/
    /**
    * @dev This function sents the owner and userContract address
    * @param _user is the UserContract.sol address
    */
    constructor(address _user)public {
    	tellorUserContract = UserContract(_user);
    	owner = msg.sender;
    }


    /*Functions*/
    /**
    * @dev Allows the user to get the latest value for the requestId specified
    * @param _requestId is the requestId to look up the value for
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp
    */
	function getCurrentValue(uint _requestId) public view returns(bool ifRetrieve, uint value, uint _timestampRetrieved) {
		TellorMaster _tellor = TellorMaster(tellorUserContract.tellorStorageAddress());
		uint _count = _tellor.getNewValueCountbyRequestId(_requestId) ;
		if(_count > 0){
				_timestampRetrieved = _tellor.getTimestampbyRequestIDandIndex(_requestId,_count -1);//will this work with a zero index? (or insta hit?)
				return(true,_tellor.retrieveData(_requestId,_timestampRetrieved),_timestampRetrieved);
        }
        return(false,0,0);
	}


	//How can we make this one more efficient?
	/**
    * @dev Allows the user to get the first verified value for the requestId after the specified timestamp
    * @param _requestId is the requestId to look up the value for
    * @param _timestamp after which to search for first verified value
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp, the timestamp after
    * which it searched for the first verified value
    */
	function getFirstVerifiedDataAfter(uint _requestId, uint _timestamp) public returns(bool,uint,uint _timestampRetrieved) {
		TellorMaster _tellor = TellorMaster(tellorUserContract.tellorStorageAddress());
		uint _count = _tellor.getNewValueCountbyRequestId(_requestId);
		if(_count > 0){
				for(uint i = _count;i > 0;i--){
					if(_tellor.getTimestampbyRequestIDandIndex(_requestId,i-1) > _timestamp && _tellor.getTimestampbyRequestIDandIndex(_requestId,i-1) < block.timestamp - 86400){
						_timestampRetrieved = _tellor.getTimestampbyRequestIDandIndex(_requestId,i-1);//will this work with a zero index? (or insta hit?)
					}
				}
				if(_timestampRetrieved > 0){
					return(true,_tellor.retrieveData(_requestId,_timestampRetrieved),_timestampRetrieved);
				}
        }
        return(false,0,0);
	}
	
	event Print(string _s,uint _num);


	/**
    * @dev Allows the user to get the first value for the requestId after the specified timestamp
    * @param _requestId is the requestId to look up the value for
    * @param _timestamp after which to search for first verified value
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp
    */
	function getAnyDataAfter(uint _requestId, uint _timestamp) public  returns(bool _ifRetrieve, uint _value, uint _timestampRetrieved){
		TellorMaster _tellor = TellorMaster(tellorUserContract.tellorStorageAddress());
		uint _count = _tellor.getNewValueCountbyRequestId(_requestId) ;
		if(_count > 0){
				emit Print("count",_count);
				for(uint i = _count;i > 0;i--){
					emit Print('tester',_tellor.getTimestampbyRequestIDandIndex(_requestId,i-1));
					emit Print('actual', _timestamp);

					if(_tellor.getTimestampbyRequestIDandIndex(_requestId,i-1) >= _timestamp){
						_timestampRetrieved = _tellor.getTimestampbyRequestIDandIndex(_requestId,i-1);//will this work with a zero index? (or insta hit?)
						emit Print("_timestampRetrieved",_timestampRetrieved);
						emit Print("value",_tellor.retrieveData(_requestId,_timestampRetrieved));
					}
				}
				if(_timestampRetrieved > 0){
					return(true,_tellor.retrieveData(_requestId,_timestampRetrieved),_timestampRetrieved);
				}
        }
        return(false,0,0);
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
	function requestData(string calldata _request,string calldata _symbol,uint _granularity, uint _tip) external{
		Tellor _tellor = Tellor(tellorUserContract.tellorStorageAddress());
		if(_tip > 0){
			require(_tellor.transferFrom(msg.sender,address(this),_tip));
		}
		_tellor.requestData(_request,_symbol,_granularity,_tip);
	}


    /**
    * @dev Allows the user to submit a request for data to the oracle using ETH
    * @param _request string API being requested to be mined
    * @param _symbol is the short string symbol for the api request
    * @param _granularity is the number of decimals miners should include on the submitted value
    * @param _tip amount the requester is willing to pay to be get on queue. Miners
    * mine the onDeckQueryHash, or the api with the highest payout pool
    */
	function requestDataWithEther(string calldata _request,string calldata _symbol,uint _granularity, uint _tip) payable external{
		tellorUserContract.requestDataWithEther.value(msg.value)(_request,_symbol,_granularity,_tip);
	}


    /** 
    * @dev Allows the user to tip miners for the specified request using Tributes
    * @param _requestId to tip
    * @param _tip amount
    */
	function addTip(uint _requestId, uint _tip) public {
		Tellor _tellor = Tellor(tellorUserContract.tellorStorageAddress());
		require(_tellor.transferFrom(msg.sender,address(this),_tip));
		_tellor.addTip(_requestId,_tip);
	}


    /**
    * @dev Allows user to add tip with Ether by sending the ETH to the userContract and exchanging it for Tributes
    * at the price specified by the userContract owner.
    * @param _requestId to tip
    * @param _tip amount
    */
	function addTipWithEther(uint _requestId, uint _tip) public payable {
		UserContract(tellorUserContract).addTipWithEther.value(msg.value)(_requestId,_tip);
	}


    /**
    * @dev allows owner to set the user contract address
    * @param _userContract address
    */
	function setUserContract(address _userContract) public {
		require(msg.sender == owner);//who should this be?
		tellorUserContract = UserContract(_userContract);
	}


    /**
    * @dev allows owner to transfer ownership
    * @param _newOwner address
    */
	function transferOwnership(address payable _newOwner) external {
            require(msg.sender == owner);
            emit OwnershipTransferred(owner, _newOwner);
            owner = _newOwner;
    }
}

```
</details>
<br>

## Instructions

Import UsingTellor.sol into your smart contract and ensure your contract inherits from it by adding "is UsingTellor".

Through your contract's constructor function pass throught the UserContract.sol address to the UsingTellor.sol contract similar to the contstructor function shown below. 

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

Now you can access all the functions within UsingTellor to requestData, addTip, retreiveData, getLastQuery by using Tributes or Ether


## Example of using Tellor as a dispute mechanism

To implement Tellor as a dispute mechanism, you need a contract that will read data off of the Tellor Oracle System.  In this example, we use two contracts along with the address for the UserContract and Using_Tellor.

* Reader.sol -- Is a test contract where we create a test contract and settle it using the Optimistic.sol oracle contract. However, for production, you would replace these two function with real (your) functions that need to read data off of the Tellor Oracle System. 

<details>
  <summary>Click to view Reader.sol!</summary>

```solidity
pragma solidity ^0.5.0;

import './Optimistic.sol';
/**
* @title Reader
* This contracts is a pretend contract using Tellor that compares two time values
*/
contract Reader is Optimistic{

	uint public startDateTime;
	uint public endDateTime;
	uint public startValue;
	uint public endValue;
	bool public longWins;
	bool public contractEnded;
	event ContractSettled(uint _svalue, uint _evalue);

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
	constructor(address _userContract, uint _disputeFeeRequired, uint _disputePeriod, uint[] memory _requestIds, uint _granularity) 
	Optimistic(_userContract,_disputeFeeRequired,_disputePeriod, _requestIds,_granularity) public {

	}


    /**
    * @dev creates a start(now) and end time(now + duration specified) for testing a contract start and end period
    * @param _duration in seconds
    */
	function testContract(uint _duration) external {
		startDateTime = now - now % granularity;
		endDateTime = now - now % granularity + _duration;
	}


	/**
    * @dev testing fucntion that settles the contract by getting the first undisputed value after the startDateTime
    * and the first undisputed value after the end time of the contract and settleling(payin off) it.
	*/
	function settleContracts() external{
		bool _didGet;
		uint _time;
		if(getIsValue(startDateTime)){
			(_didGet, startValue, _time) = getFirstUndisputedValueAfter(startDateTime);
			if(_didGet && getIsValue(endDateTime)){
				(_didGet, endValue, _time) = getFirstUndisputedValueAfter(endDateTime);
				if(_didGet){
					if(endValue > startValue){
						longWins = true;
					}
					contractEnded = true;
					emit ContractSettled(startValue, endValue);
				}

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

import './UsingTellor.sol';
import '../TellorMaster.sol';
import '../Tellor.sol';
/**
* @title UsingTellor
* This contracts creates for easy integration to the Tellor Tellor System
*/
contract Optimistic is UsingTellor{

    //Can we rework soem of these mappings into a struct?
	mapping(uint => bool) public isValue;//mapping for timestamp to bool where it's true if the value as been set
	mapping(uint => uint) valuesByTimestamp; //mapping of timestamp to value
	mapping(uint => bool) public disputedValues;//maping of timestamp to bool where it's true if the value has been disputed
	mapping(uint => uint[]) public requestIdsIncluded; //mapping of timestamp to requestsId's to include

    // struct infoForTimestamp {
    // 	bool isValue;
    // 	uint valuesByTimestamp;
    // 	bool disputedValues;
    // 	uint[] requestIdsIncluded;
    // }
    // mapping(uint => infoForTimestamp) public infoForTimestamps;//mapping timestampt to InfoForTimestamp struct

	uint[] timestamps; //timestamps with values
	uint[] requestIds;
	uint[] disputedValuesArray;
	uint public granularity;
	uint public disputeFee; //In Tributes
	uint public disputePeriod;
	
	event NewValueSet(uint indexed _timestamp, uint _value);
	event ValueDisputed(address _disputer,uint _timestamp, uint _value);
	event TellorValuePlaced(uint _timestamp, uint _value);
	event Print(bool _call);
	event Print2(uint _1, uint _2);

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
	constructor(address _userContract, uint _disputeFeeRequired, uint _disputePeriod, uint[] memory _requestIds, uint _granularity) UsingTellor(_userContract) public{
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
	function setValue(uint _timestamp, uint _value) external{
		//Only allows owner to set value
		require(msg.sender == owner);
		//Checks that no value has already been set for the timestamp
		require(getIsValue(_timestamp) == false);
		//sets timestamp
		valuesByTimestamp[_timestamp] = _value;
		//sets isValue to true once value is set
		isValue[_timestamp] = true;
		//adds timestamp to the timestamps array
		timestamps.push(_timestamp);
		//lets the network know a new timestamp and value have been added
		emit NewValueSet(_timestamp,_value);

	}

    /**
    * @dev allows user to initiate dispute on the value of the specified timestamp
    * @param _timestamp is the timestamp for the value to be disputed
    */
	function disputeOptimisticValue(uint _timestamp) external{
		TellorMaster _tellor = TellorMaster(tellorUserContract.tellorStorageAddress());
        uint _disputeFee = _tellor.getUintVar(keccak256("currentTotalTips"));//get latest tip paid to mine
    	//Prepares the signed transaction info to transfer the dispute fee from the party calling this function
		//(msg.sender) to this contract
		bytes memory sig = abi.encodeWithSignature("transferFrom(address,address,uint256)",msg.sender,address(this),_disputeFee);
		//sets the TellorUserContract
		address addr  = tellorUserContract.tellorStorageAddress();
		//transfers the dispute fee from the party calling this function(msg.sender) to this contract
		(bool success, bytes memory data) = addr.call(sig);
		//require the tranfer fee was successful
		require(success);
		//require that isValue for the timestamp being disputed to exist/be true
		require(isValue[_timestamp]);
		// assert disputePeriod is still open
		require(now - now % granularity  <= _timestamp + disputePeriod);
        //set the disputValues for the disputed timestamp to true 
		disputedValues[_timestamp] = true;
		//add the disputed timestamp to the diputedValues array
		disputedValuesArray.push(_timestamp);
		emit ValueDisputed(msg.sender,_timestamp,valuesByTimestamp[_timestamp]);
	}


	/**
    * @dev This function gets the Tellor requestIds values for the disputed timestamp. It averages the values on the 
    * requestsIds and replaces the value set by the contract owner, centralized party.
    * @param _timestamp to get Tellor data from
    * @return uint of new value and true if it was able to get Tellor data
	*/
	function getTellorValues(uint _timestamp) public returns(uint _value, bool _didGet){
		//We need to get the tellor value within the granularity.  If no Tellor value is available...what then?  Simply put no Value?  
		//No basically, the dispute period for anyValue is within the granularity
		TellorMaster _tellor = TellorMaster(tellorUserContract.tellorStorageAddress());
		Tellor _tellorCore = Tellor(tellorUserContract.tellorStorageAddress());
		uint _retrievedTimestamp;
		uint _initialBalance = _tellor.balanceOf(address(this));//Checks the balance of Tellor Tributes on this contract
		//Loops through all the Tellor requestsId's initially(in the constructor) associated with this contract data
		for(uint i = 1; i <= requestIds.length; i++){
			//Get all values for that requestIds' timestamp
			//Check if any is after your given timestamp
			//If yes, return that value. If no, then request that Id
			(_didGet,_value,_retrievedTimestamp) = getFirstVerifiedDataAfter(i,_timestamp);
			if(_didGet){
				uint _newTime = _retrievedTimestamp - _retrievedTimestamp % granularity; //why are we using the mod granularity???
				//provides the average of the requests Ids' associated with this price feed
				uint _newValue =(_value + valuesByTimestamp[_newTime] * requestIdsIncluded[_newTime].length) / (requestIdsIncluded[_newTime].length + 1);
				//Add the new timestamp and value 
				valuesByTimestamp[_newTime] = _newValue;
				emit TellorValuePlaced(_newTime,_newValue);
				emit Print2(_newValue,_value);
				//records the requests Ids included on the price average where all prices came from Tellor requests Ids
				requestIdsIncluded[_newTime].push(i); //how do we make sure it's not called twice?
				//if the value for the newTime does not exist, then push the value, update the isValue to true
				//otherwise if the newTime is under dsipute then update the dispute status to false
				// ??? should the else be an "and"
				if(isValue[_newTime] == false){
							timestamps.push(_newTime);
							isValue[_newTime] = true;
							emit NewValueSet(_newTime,_value);
				}
				else if(disputedValues[_newTime] == true){
					disputedValues[_newTime] = false;
				}
			}
			//otherwise request the ID and split the contracts initial tributes balance to equally tip all 
			//requests Ids associated with this price feed
			else{
				if(_tellor.balanceOf(address(this)) > requestIds.length){
					//Request Id to be mined by adding to it's tip
					_tellorCore.addTip(i, _initialBalance / requestIds.length);
				}
			}
		}
	}


    /**
    * @dev Get the first undisputed value after the timestamp specified. This function is used within the getTellorValues
    * but can be used on its own. 
    * @param _timestamp to search the first undisputed value there after
    */
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


    /*Getters*/
    /**
    * @dev Getter function for the value based on the timestamp specified
    * @param _timestamp to retreive value from
    */
	function getMyValuesByTimestamp(uint _timestamp) public view returns(uint value){
		return valuesByTimestamp[_timestamp];
	}


    /**
    * @dev Getter function for the number of RequestIds associated with a timestamp, based on the timestamp specified
    * @param _timestamp to retreive number of requestIds
    * @return uint count of number of values for the spedified timestamp
    */
	function getNumberOfValuesPerTimestamp(uint _timestamp) external view returns(uint){
			return requestIdsIncluded[_timestamp].length;
	}


    /**
    * @dev Checks to if a value exists for the specifived timestamp
    * @param _timestamp to verify
    * @return ture if it exists
    */
	function getIsValue(uint _timestamp) public view returns(bool){
		return isValue[_timestamp];
	}


    /**
    * @dev Getter function for latest value available
    * @return latest value available
    */
	function getCurrentValue() external view returns(uint){
	    require(timestamps.length > 0);
		return getMyValuesByTimestamp(timestamps[timestamps.length -1]);
	}

    /**
    * @dev Getter function for the timestamps available
    * @return uint array of timestamps available
    */
	function getTimestamps() external view returns(uint[] memory){
		return timestamps;
	}


    /**
    * @dev Getter function for the requests Ids' from Tellor associated with this price feed
    * @return uint array of requests Ids'
    */
	function getRequestIds() external view returns(uint[] memory){
		return requestIds;
	}


    /**
    * @dev Getter function for the requests Ids' from Tellor associated with this price feed
    * at the specified timestamp. This only gets populated after a dispute is initiated and the 
    * function getTellorValues is ran.
    * @param _timestamp to retreive the requestIds
    * @return uint array of requests Ids' included in the calcluation of the value
    */
	function getRequestIdsIncluded(uint _timestamp) external view returns(uint[] memory){
		return requestIdsIncluded[_timestamp];
	}


    /**
    * @dev Getter function for the number of disputed values 
    * @return uint count of number of values for the spedified timestamp
    */
	function getNumberOfDisputedValues() external view returns(uint){
		return disputedValuesArray.length;
	}


    /**
    * @dev Getter function for all disputed values
    * @return the array with all values under dispute
    */
	function getDisputedValues() external view returns(uint[] memory){
		return disputedValuesArray;
	}


    /**
    * @dev This checks if the value for the specified timestamp is under dispute 
    * @param _timestamp to check if it is under dispute
    * @return true if it is under dispute
    */
	function isDisputed(uint _timestamp) external view returns(bool){
		return disputedValues[_timestamp];
	}


    /**
    * @dev Getter function for the dispute value by index
    * @return the value
    */
	function getDisputedValueByIndex(uint _index) external view returns(uint){
		return disputedValuesArray[_index];
	}

}

```
</details>
<br>

<b>Instructions:</b> 
Deploy the Reader contract and link it to the UserContract. On the Constructor, specifiy the UserContract Address, disputeFeeRequired, disputePeriod, all requestIds from Tellor that correspond to the price feed needed(the function averages all of these to determine the "correct" price), and granularity(the number of decimals to be included on the price value). 


```solidity
contract Reader is Optimistic{

constructor(address _userContract, uint _disputeFeeRequired, uint _disputePeriod, uint[] memory _requestIds, uint _granularity) Optimistic(_userContract,_disputeFeeRequired,_disputePeriod, _requestIds,_granularity) public {}

```

