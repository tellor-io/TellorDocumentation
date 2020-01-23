# Run TellorMiner from Latest Binary Release

This is the workhorse of the Miner system as it takes on solving the PoW challenge.  

It's built on Go and utilizes a split structure.  The database piece is a LevelDB that keeps track of all variables (challenges, difficulty, values to submit, etc.) and the miner simply solves the PoW challenge.  This enables parties to split the pieces for optimization.

## Download the Latest Binary Release

#### Linux
```
wget https://github.com/tellor-io/TellorMiner/releases/latest/download/TellorMiner
```
#### Windows

Download executable file: 

[https://github.com/tellor-io/TellorMiner/releases/latest/download/TellorMiner.exe](https://github.com/tellor-io/TellorMiner/releases/latest/download/TellorMiner.exe)


## Update the Miner Configuration File
Start by downloading the sample configuration file:
```
wget https://raw.githubusercontent.com/tellor-io/TellorMiner/master/config.json
```
Now, edit the `config.json`, be sure to update these values:
1. Set `nodeUrl` to an Ethereum node endpoint (e.g. Infura API endpoint)
2. Set `privateKey` to the private key for the Ethereum wallet you plan to use
3. Set `publicAddress` to the public key for the Ethereum wallet you plan to use (note no "0x" prefix)
4. Set `serverWhitelist` so that it includes your `publicAddress` (this whitelists your miner to use your local dataServer)
5. Add `requestData` and set the value to `0` so you're not constantly requesting data from Tellor

### Utilizing your GPU
Your GPU is enabled by default, but to edit any of the specifics of the configuration, edit the following line in your `config.json` file:
 
        "gpuConfig":{
            "default":{
                "groupSize":256,
                "groups":4096,
                "count":16
            }
        }

If you would like to specify different config settings for each GPU card, you can use the GPU's name in the following format:

        "gpuConfig":{
            "<GPUName1>":{
                "groupSize":256,
                "groups":4096,
                "count":16
            },
            "<GPUName2>":{
                "groupSize":256,
                "groups":4096,
                "count":16
            }, 
            "<GPUName3>":{
                "groupSize":256,
                "groups":4096,
                "count":16,
                "disabled":true
            }, 
            
        }


If you wish to tweak the variables for performance, do so at your own risk:

			groupSize - number of groups to split work into
			groups - number of groups of work submitted to the gpu at once. needs to be large enough to fully load the gpu. different numbers here can produce drastically different hash rates
			count: number of hashes each thread executes in one pass
            disabled: boolean on whether or not to disable a specific GPU


## Download the PSR and Logging Config Files
The `psr.json` file contains information about the different request and associated URLs. Download the latest version:
```
wget https://raw.githubusercontent.com/tellor-io/TellorMiner/master/psr.json

```
The `loggingConfig.json` file contains information about the levels of logging in your terminal. Download the latest version:
```
wget https://raw.githubusercontent.com/tellor-io/TellorMiner/master/loggingConfig.json

```
## Start Mining
Tellor is a staked miner. You will need 1000 TRB to mine. Additionally, TellorMiner requires that you run a dataServer process and a miner process. The instructions below can be used to get started.

### Deposit your initial stake
To deposit your stake you can run the following command:
```
./TellorMiner -deposit -config=./config.json -psrPath=./psr.json
```

### Run the Miner
Start the miner by running this command in another terminal or process:
```
./TellorMiner -miner -dataServer -config=./config.json -psrPath=./psr.json
```
After starting the miner, observe the logs it outputs to confirm it's working correctly.

## TellorMiner Reference
Here is some information about the TellorMiner for reference.

### Required Flags
* **-psrPath** (path to your PSR file)
* **-config** (path to your config file)

### Optional Flags
* **-miner** (indicates to run the miner)
* **-dataServer** (indicates to run the dataServer)
* **-transfer -toAddress -amount** (do not run these with miner/dataServer)(indicates transfer, toAddress is Ethereum address and amount is number of Tributes (note 18 decimals))
* **-deposit** (do not run these with miner/dataServer)(indicates to deposit 1000 tokens in the contract (note you must have 1000 Tributes in your account))
* **-logConfig** (path to loggingConfig file if you want to save log output)
* **-requestStakingWithdraw** (indicates you wish to withdraw your stake)
* **-withdraw** (withdraws your stake, run 1 week after request)


### Config file options:
```
* contractAddress (required) - address of TellorContract
* nodeURL (required) - node URL (e.g https://mainnet.infura.io/bbbb or https://localhost:8545 if own node)
* privateKey (required) - privateKey for your address (note, no 0x)
* databaseURL (required) - where you are reading from for the server database (if hosted)
* publicAddress (required) - public address for your miner (note, no 0x)
* ethClientTimeout (required) - timeout for making requests from your node
* trackerSleepCycle (required) - how often your database updates
* trackers (required) - which pieces of the database you update
* dbFile (required) - where you want to store your local database (if self-hosting)
* serverHost (required) - location to host server
* serverPort (required) - port to host server
* serverWhitelist (required) - whitelists which publicAddress can access the data server
* fetchTimeout - timeout for requesting data from an API
* requestData - Will your miner request Data if challenge is 0.  If yes, then you will addTip() to this number.  Enter a uint number representing request id to be requested (e.g. 2)
* gasMultiplier - Multiplies the submitted gasPrice (e.g. 2 (will double gas costs))
* gasMax - a max for the gas price in gwei (note: this max comes BEFORE the gas multiplier.  So a max gas cost of 10 gwei, can have gas prices up to 20 if gasMultiplier is 2)
* heartbeat - an integer that controls how frequently the miner process should report the hashrate (larger is less frequent, try 1000000 to start)
* numProcessors - an integer number of processors to use for mining
* useGPU - a boolean as to whether to use the GPU on your machine.  Will use the CPU if false
```

### LogConfig file options:

The logging.config file consists of two fields:
* component
* level


The component is the package.component combination.  
E.G. the Runner component in the tracker package would be:
tracker.Runner

To turn on logging, add the component and the according level.  Note the default level is "INFO", so to turn down the number of logs, enter "WARN" or "ERROR"

```
DEBUG - logs everything in INFO and additional developer logs
INFO - logs most information about the mining operation
WARN - logs all warnings and errors
ERROR - logs only serious errors
```

### Running a remote data server

If you are running multiple miners, there is no reason to run multiple databases (the values you will submit should be identical).  In addition, querying the same API from multiple processes can lead to rate limits on the public API's.  To get around this, you can utilize a system where you run one:

    TellorMiner -dataServer

and multiple miners that all read off of the one database

        TellorMiner -miner -config=./config1
        TellorMiner -miner -config=./config2 
        etc..

For instructions:
https://docs.google.com/document/d/1k8ELb1cXkEpztHkHUt8QTL4JCcnHw5_yQjTKIHCaSCE

## Connecting to a pool

Add the following to your config file: 

    "enablePoolWorker": true,
    "poolURL": "<poolURL>",

Where the <poolURL> is the link to your pool. Current tellor pools:

        http://tellorpool.org

Further options:

    "poolJobDuration":10 

The job duration is the time in seconds to grab information from the pool.  The default time is 15 seconds. 


## Ending Mining Operations / Unstaking

To unstake your tokens, you need to request a withdraw: 
```
./TellorMiner requestStakingWithdraw -config=./config.json -psrPath=./psr.json
```
One week after the request, the tokens are free to move at your discretion after running the command:

```
./TellorMiner -withdraw -config=./config.json -psrPath=./psr.json
```

### DISCLAIMER


    Mine at your own risk.  

    Mining requires you deposit 1000 Tellor Tributes.  These are a security deposity.  If you are a malicious actor (aka submit a bad value), the community can vote to slash your 1000 tokens.  

    Mining also requires submitting on-chain transactions on Ethereum.  These transactions cost gas (ETH) and can sometimes be signifiant if the cost of gas on EThereum is high (i.e. the network is clogged).  Please reach out to the community to find the best tips for keeping gas costs under control or at least being aware of the costs. 

    If you are building a competing client, please contact us.  A lot of the miner specifications are off-chain and a significant portion of the mining process hinges on the consensus of the Tellor community to determine what proper values are.  Competing clients that change different pieces run the risk of being disputed by the commmunity.  

    There is no guaruntee of profit from mining. 

    There is no promise that Tellor Tributes currently hold or will ever hold any value. 