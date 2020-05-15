## Integrate Tellor

To integrate Tellor as your smart contracts' source of data you need to integrate the user contract, UsingTellor.sol into your smart contract.

UsingTellor.sol allows your contract to easily read data from Tellor. 

## Quick install with npm
To quickly install Tellor for testing into your project run the following command: 

   npm install usingtellor


## Sample Repo

[https://github.com/tellor-io/sampleUsingTellor](https://github.com/tellor-io/sampleUsingTellor)

## Using Tellor Descriptions

<br>
<details>
  <summary>Click to view UsingTellor.sol!</summary>

```solidity
pragma solidity ^0.5.0;

import "../contracts/testContracts/TellorMaster.sol";
import "./OracleIDDescriptions.sol";
import "../contracts/interfaces/EIP2362Interface.sol";
import "../contracts/libraries/TellorLibrary.sol";//imported for testing ease
import "../contracts/testContracts/Tellor.sol";//imported for testing ease

/**
* @title UserContract
* This contracts creates for easy integration to the Tellor System
* by allowing smart contracts to read data off Tellor
*/
contract UsingTellor is EIP2362Interface{
    address payable public tellorStorageAddress;
    address public oracleIDDescriptionsAddress;
    TellorMaster _tellorm;
    OracleIDDescriptions descriptions;

    event NewDescriptorSet(address _descriptorSet);

    /*Constructor*/
    /**
    * @dev the constructor sets the storage address and owner
    * @param _storage is the TellorMaster address
    */
    constructor(address payable _storage) public {
        tellorStorageAddress = _storage;
        _tellorm = TellorMaster(tellorStorageAddress);
    }

    /*Functions*/
    /*
    * @dev Allows the owner to set the address for the oracleID descriptors
    * used by the ADO members for price key value pairs standarization 
    * _oracleDescriptors is the address for the OracleIDDescriptions contract
    */
    function setOracleIDDescriptors(address _oracleDescriptors) external {
        require(oracleIDDescriptionsAddress == address(0), "Already Set");
        oracleIDDescriptionsAddress = _oracleDescriptors;
        descriptions = OracleIDDescriptions(_oracleDescriptors);
        emit NewDescriptorSet(_oracleDescriptors);
    }

    /**
    * @dev Allows the user to get the latest value for the requestId specified
    * @param _requestId is the requestId to look up the value for
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp
    */
    function getCurrentValue(uint256 _requestId) public view returns (bool ifRetrieve, uint256 value, uint256 _timestampRetrieved) {
        return getDataBefore(_requestId,now);
    }

    /**
    * @dev Allows the user to get the latest value for the requestId specified using the 
    * ADO specification for the standard inteface for price oracles
    * @param _bytesId is the ADO standarized bytes32 price/key value pair identifier
    * @return the timestamp, outcome or value/ and the status code (for retreived, null, etc...)
    */
    function valueFor(bytes32 _bytesId) view external returns (int value, uint256 timestamp, uint status) {
        uint _id = descriptions.getTellorIdFromBytes(_bytesId);
        int n = descriptions.getGranularityAdjFactor(_bytesId);
        if (_id > 0){
            bool _didGet;
            uint256 _returnedValue;
            uint256 _timestampRetrieved;
            (_didGet,_returnedValue,_timestampRetrieved) = getDataBefore(_id,now);
            if(_didGet){
                return (int(_returnedValue)*n,_timestampRetrieved, descriptions.getStatusFromTellorStatus(1));
            }
            else{
                return (0,0,descriptions.getStatusFromTellorStatus(2));
            }
        }
        return (0, 0, descriptions.getStatusFromTellorStatus(0));
    }

    /**
    * @dev Allows the user to get the first value for the requestId after the specified timestamp
    * @param _requestId is the requestId to look up the value for
    * @param _timestamp after which to search for first verified value
    * @return bool true if it is able to retreive a value, the value, and the value's timestamp
    */
    function getDataBefore(uint256 _requestId, uint256 _timestamp)
        public
        view
        returns (bool _ifRetrieve, uint256 _value, uint256 _timestampRetrieved)
    {
        uint256 _count = _tellorm.getNewValueCountbyRequestId(_requestId);
        if (_count > 0) {
            for (uint256 i = 1; i <= _count; i++) {
                uint256 _time = _tellorm.getTimestampbyRequestIDandIndex(_requestId, i - 1);
                if (_time <= _timestamp && _tellorm.isInDispute(_requestId,_time) == false) {
                    _timestampRetrieved = _time;
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

## Instructions

Import UsingTellor.sol into your smart contract and ensure your contract inherits from it by adding "is UsingTellor".

Through your contract's constructor function pass through the tellor address to the UsingTellor.sol contract similar to the contstructor function shown below.

You'll also need to know what tellor ID you are querying in order to get the correct data.  For a list of what data is available, you can check here:
[https://docs.google.com/spreadsheets/d/1rRRklc4_LvzJFCHqIgiiNEc7eo_MUw3NRvYmh1HyV14](https://docs.google.com/spreadsheets/d/1rRRklc4_LvzJFCHqIgiiNEc7eo_MUw3NRvYmh1HyV14)

Mainnet Address- [0x0ba45a8b5d5575935b8158a88c631e9f9c95a2e5](https://etherscan.io/address/0x0ba45a8b5d5575935b8158a88c631e9f9c95a2e5)

Rinkeby Address- [0xFe41Cb708CD98C5B20423433309E55b53F79134a](https://rinkeby.etherscan.io/address/0xFe41Cb708CD98C5B20423433309E55b53F79134a)

```solidity
  constructor(uint _tellorID, address payable _tellorAddress) UsingTellor(_tellorAddress) public {
    tellorID = _tellorID;
  }

```

Now you have access to three functions: 

```
    //retrieves current value for the id
    function getCurrentValue(uint256 _requestId) public view returns (bool ifRetrieve, uint256 value, uint256 _timestampRetrieved)

    //retrieves current value in EIP2362 format
    function valueFor(bytes32 _bytesId) view external returns (int value, uint256 timestamp, uint status)

    //retrieves qualified data before (e.g. now - 1 hours) in order to wait for disputes/checks
    function getDataBefore(uint256 _requestId, uint256 _timestamp) public view returns (bool _ifRetrieve, uint256 _value, uint256 _timestampRetrieved)

```


<b>Instructions:</b> 

###Tellor Migrations file:

* To using truffle migrate with your files you will need a custom migrations file to deploy Tellor along with your contracts.  

An example (deploying the sampleUsingTellor contract) can be found here:
[https://github.com/tellor-io/sampleUsingTellor/blob/master/migrations/2_tellor_migration.js](https://github.com/tellor-io/sampleUsingTellor/blob/master/migrations/2_tellor_migration.js)


### Testing

Testing locally is possible because usingTellor includes a mock tellor system which will you allow you to deploy a fake Tellor and mine set values for testing.

An example test can be found here:
[https://github.com/tellor-io/sampleUsingTellor/blob/master/test/testSampleTellor.js](https://github.com/tellor-io/sampleUsingTellor/blob/master/test/testSampleTellor.js)

Testing on rinkeby will require you to get rinkeby TRB for data requests. Please reach out to us on discord, Telegram, or at [info@tellor.io](info@tellor.io) and we'll send you some.  


### Verification

The usingTellor repo also contains flat files which contain the necessary usingTellor pieces to add to your contracts file.  

It can be found here:  [https://github.com/tellor-io/usingtellor/tree/master/flatfiles](https://github.com/tellor-io/usingtellor/tree/master/flatfiles)



