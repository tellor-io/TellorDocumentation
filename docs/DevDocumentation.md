<!---
<span style="color:#06D88C"> Tellor </span>
-->

## Contracts Description <a name="Contracts-Description"> </a>
* <b>Tellor.sol</b> -- is the Tellor oracle contract and it allows miners to submit the proof of work, requestId, and value, sorts the values, pays the miners, allows the data users to request data and "tip" the miners to incentivize them to provide values, allows the users to retrieve and dispute the values.
    * <b>Tellor.sol</b> --contains all the functions
       * <b>TellorLibrary.sol</b> --contains the logic for the functions in Tellor.sol
    * <b>TellorMaster.sol</b> -- contains the delegate calls to allow Tellor.sol to write to the TellorGetters.sol. TellorMaster is TellorGetters.sol
        * <b>TellorGetters.sol</b> -- stores all the Tellor.sol variables 
        * <b>TellorGettersLibrary.sol</b> --contains the logic for the functions in TellorGetters.sol


## Scripts Description <a name="Scripts-Description"> </a>

* <b>01_DeployTellor.js</b> -- deploys and connects Tellor.sol and TellorMaster.sol


## Deploy Tellor<a name="operator-setup"> </a>  
Tellor can be deployed using the 01_DeployTellor.js script and folowing the [Instructions for quick start with Truffle Deployment](#Quick-Deployment) . However, the process is documented below for deploying through any environment. 

The following documentation is noted for acting as the operator. Specific contract details are laid out for ease of use regardless of dev environment. 

**Step 1: Operator - Deploy Tellor.sol**  
The first deployed Tellor.sol.

```solidity
Tellor();
TellorMaster(Tellor.address); //where the tellor.address is the address of the deployed Tellor() above.
```

<!---

  $ npm install tellor

On contracts use “is usingTellor” to access these functions: requestData, retreiveData,  getLastQuery.
-->

## Instructions for quick start with Truffle Deployment <a name="Quick-Deployment"> </a> 
Follow the steps below to launch the Oracle contracts using Truffle. 

*  Open two terminals.
*  On one terminal run:
    Clone the repo, cd into it, and then run:
    $ npm install
    $ truffle compile
    $ truffle migrate
    $ truffle exec scripts/01_DeployTellor.js

### Testing through Truffle<a name="testing"> </a>
*  On the second termial run:
```solidity
   $ ganache-cli -m "nick lucian brenda kevin sam fiscal patch fly damp ocean produce wish"
```
*  On the first terminal run: 

```solidity
    $   truffle test
```
*  And wait for the message 'START MINING RIG!!'
*  Kick off the python miner file [./miner/testMinerB.py](https://github.com/tellor-io/TellorCore/tree/master/miner/testMinerB.py).


Production and test python miners are available under the miner subdirectory [here](https://github.com/tellor-io/TellorCore/tree/master/miner). You will need to get at least 5 miners running.

Step by step instructions on setting up a Tellor Oracle without truffle are available here: [Detailed documentation for self setup](https://tellor.readthedocs.io/en/latest/DevDocumentation/)

