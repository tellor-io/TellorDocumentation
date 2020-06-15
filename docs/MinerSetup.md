# Run TellorMiner from Latest Binary Release

This is the workhorse of the Miner system as it takes on solving the PoW challenge.  

It's built on Go and utilizes a split structure.  The database piece is a LevelDB that keeps track of all variables (challenges, difficulty, values to submit, etc.) and the miner simply solves the PoW challenge.  This enables parties to split the pieces for optimization.

**The Tellor system is a way to push data on-chain.  What the pieces of data are are specificied in the psr.json file. Note that the data corresponds to a specific API.  The tellor mining system is set up to pull api data to generate these values to submit on-chain once a correct nonce is mined. These specific apis are just suggestions.** 

**The system is not guaranteed to work for everyone.  It is up to the consensus of the Tellor token holders to determine what a correct value is. As an example, request ID 4 is BTC/USD.  If the api's all go down, it is the responsibility of the miner to still submit a valid BTC/USD price.  If they do not, they risk being disputed and slashed.  For these reasons, please contribute openly to the official Tellor miner (or an open source variant), as consensus here is key.  If you're miner gets a different value than the majority of the of the other miners, you risk being punished.**

A list of all PSR's and the data expected can be found here: [https://docs.google.com/spreadsheets/d/1rRRklc4_LvzJFCHqIgiiNEc7eo_MUw3NRvYmh1HyV14](https://docs.google.com/spreadsheets/d/1rRRklc4_LvzJFCHqIgiiNEc7eo_MUw3NRvYmh1HyV14)


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

2. Set `publicAddress` to the public key for the Ethereum wallet you plan to use (note no "0x" prefix)

3. Set `serverWhitelist` so that it includes your `publicAddress` (this whitelists your miner to use your local dataServer)

4. Add `requestData` and set the value to `0` so you're not constantly requesting data from Tellor

## Create .env file

Create a file named .env and put your private key in it. Example:

```
ETH_PRIVATE_KEY="3a10b4bc1258e8bfefb95b498fb8c0f0cd6964a811eabca87df56xxxxxxxxxxxx"
```

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
                "disabled":true
            }, 
            
        }


If you wish to tweak the variables for performance, do so at your own risk:

			groupSize - number of groups to split work into
			groups - number of groups of work submitted to the gpu at once. needs to be large enough to fully load the gpu. different numbers here can produce drastically different hash rates
			count: number of hashes each thread executes in one pass
            disabled: boolean on whether or not to disable a specific GPU


## Download the API list and Logging Config Files
The `indexes.json` file contains information about the different API's that you query for price information.  Download the latest version:
```
- wget https://raw.githubusercontent.com/tellor-io/TellorMiner/dev/indexes.json

```
If you would like to add API's or change the API's specified, feel free to add to or change those listed in the file. (e.g. if you can't query binance api's, you can change them out for an autheticated API call (with parsing) to a different exchange). 


The `loggingConfig.json` file contains information about the levels of logging in your terminal. Download the latest version:
```
wget https://raw.githubusercontent.com/tellor-io/TellorMiner/master/loggingConfig.json

```

## Manual Data Entry

Tellor currently has one data point which must be manually created.  The rolling 3 month average of the US PCE .  Updated monthly:  [https://apps.bea.gov/iTable/iTable.cfm?reqid=19&step=3&isuri=1&1921=survey&1903=84#reqid=19&step=3&isuri=1&1921=survey&1903=84]( https://apps.bea.gov/iTable/iTable.cfm?reqid=19&step=3&isuri=1&1921=survey&1903=84#reqid=19&step=3&isuri=1&1921=survey&1903=84)

```
wget https://raw.githubusercontent.com/tellor-io/TellorMiner/dev/manualData.json
```

## Start Mining
Tellor is a staked miner. You will need 1000 TRB to mine. Additionally, TellorMiner requires that you run a dataServer process and a miner process. The instructions below can be used to get started.

### Deposit your initial stake
To deposit your stake you can run the following command:
```
./TellorMiner --config=./config.json stake deposit
```

### Run the Miner
Start the miner by running this command in another terminal or process:
```
./TellorMiner --config=./config.json mine
```
After starting the miner, observe the logs it outputs to confirm it's working correctly.

## TellorMiner Reference
Here is some information about the TellorMiner for reference.

### Required Flags
* **--config** (path to your config file)


### Commands
* **--logConfig** (location of logging config file; default path is current directory)
* **mine** (indicates to run the miner)
* **mine -r** (indicates to mine utilizing a remote server)
* **dataserver** (indicates to run the dataServer (no mining))
* **transfer (AMOUNT) (TOADDRESS)**  (indicates transfer, toAddress is Ethereum address and amount is number of Tributes (eg. transfer 10 0xea... (this transfers 10 tokens)))
* **approve (AMOUNT) (TOADDRESS)** (ammount to approve the toaddress to send this amount of tokens
* **stake deposit** (indicates to deposit 1000 tokens in the contract (note you must have 1000 Tributes in your account))
* **stake request** (indicates you wish to withdraw your stake)
* **stake withdraw** (withdraws your stake, run 1 week after request)
* **stake status** (shows your staking balance)
* **balance** (shows your balance)


### Config file options:
```
* contractAddress (required) - address of TellorContract
* nodeURL (required) - node URL (e.g https://mainnet.infura.io/bbbb or https://localhost:8545 if own node)
* privateKey (required) - privateKey for your address (note, no 0x)
* databaseURL (required) - where you are reading from for the server database (if hosted)
* publicAddress (required) - public address for your miner (note, no 0x)
* ethClientTimeout (required) - timeout for making requests from your node
* trackerCycle (required) - how often your database updates (in seconds)
* trackers (required) - which pieces of the database you update
* dbFile (required) - where you want to store your local database (if self-hosting)
* serverHost (required) - location to host server
* serverPort (required) - port to host server
* serverWhitelist (required) - whitelists which publicAddress can access the data server
* fetchTimeout - timeout for requesting data from an API
* requestData - Will your miner request Data if challenge is 0.  If yes, then you will addTip() to this number.  Enter a uint number representing request id to be requested (e.g. 2)
* requestDataInterval - min frequency at which to request data at (in seconds, default 30)
* gasMultiplier - Multiplies the submitted gasPrice (e.g. 2 (will double gas costs))
* gasMax - a max for the gas price in gwei (note: this max comes BEFORE the gas multiplier.  So a max gas cost of 10 gwei, can have gas prices up to 20 if gasMultiplier is 2)
* heartbeat - an integer that controls how frequently the miner process should report the hashrate (larger is less frequent, try 1000000 to start)
* numProcessors - an integer number of processors to use for mining
* gpuConfig:{
        "foo":{
                "groupSize":64,
                "groups":128,
                "count":256,
                "disabled":false
        }
    }  -  GPU config options
* disputeTimeDelta - how far back to store values for min/max range - default 5 (in minutes)
* disputeThreshold - percentage of acceptable range outside min/max for dispute checking - default 0.01 (1%)
* psrFolder - folder location holding your psr.json file, default working directory

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

    TellorMiner dataServer

and multiple miners that all read off of the one database

        TellorMiner mine -r
        TellorMiner mine -r
        etc..

To do this you need to edit your config file for the dataServer to add the public addresses of all remote miners that want to read the database:

            “serverWhitelist”: [
                    "0x1...",
                    "0x2...."
            ]

And then in both your miner and dataServer config files, you must change the IP addresses to match (the server/port you are hosting/reading from)

        "serverHost": "1.2.3.4",
        "serverPort": 5000,

Note that your dataServer and miners must be separate commands


For more detailed instructions:
[https://docs.google.com/document/d/1k8ELb1cXkEpztHkHUt8QTL4JCcnHw5_yQjTKIHCaSCE](https://docs.google.com/document/d/1k8ELb1cXkEpztHkHUt8QTL4JCcnHw5_yQjTKIHCaSCE)

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
./TellorMiner --config=./config.json stake request
```
One week after the request, the tokens are free to move at your discretion after running the command:

```
./TellorMiner --config=./config.json stake withdraw
```

## Running the Disputer

Tellor as a system only functions properly if parties actively monitor the tellor network and dispute bad values.  Expecting parties to manually look at every value submitted is obviously burdensome.  The Tellor disputer automates this fact checking of values.  

The way that it work is that your dataServer will store historical values (e.g. the last 10 minutes) and then compare any submitted values to the min/max of the historical values.  If the value submitted is outside a certain threshold (e.g. 10% of the min/max), then the party will be notified and they can choose if they wish to dispute the bad value.  

To start the disputer, add the following line to your config file IN THE TRACKERS ARRAY:

    "disputeChecker"

Now when running the dataServer, you will store historical values and check for whether the submitted values were within min/max of the range of historical values given a threshold (e.g. 1% outside).  The variables for configuring the time range of the historical values and the threshold are as follows:


        disputeTimeDelta: 5,
        disputeThreshold: 0.01,

Where 5 and .01 are the defaults, the variables are the amount of time in minutes to store historical values for comparison and the the threshold outside the min/max of the values (e.g. 0.01 = 1%);


If the disputer is succesful and finds a submitted outside of your acceptable range, a text file containing pertinent information will be created in your working directory (the one you're running the miner out of) in the format: 

        "possible-dispute-(blocktime).txt"


### DISCLAIMER


    Mine at your own risk.  

    Mining requires you deposit 1000 Tellor Tributes.  These are a security deposity.  If you are a malicious actor (aka submit a bad value), the community can vote to slash your 1000 tokens.  

    Mining also requires submitting on-chain transactions on Ethereum.  These transactions cost gas (ETH) and can sometimes be signifiant if the cost of gas on EThereum is high (i.e. the network is clogged).  Please reach out to the community to find the best tips for keeping gas costs under control or at least being aware of the costs. 

    If you are building a competing client, please contact us.  A lot of the miner specifications are off-chain and a significant portion of the mining process hinges on the consensus of the Tellor community to determine what proper values are.  Competing clients that change different pieces run the risk of being disputed by the commmunity.  

    There is no guaruntee of profit from mining. 

    There is no promise that Tellor Tributes currently hold or will ever hold any value. 