# Run TellorMiner from Latest Binary Release
These instructions are for installing and running TellorMiner from source on Linux. These have been tested on Ubuntu 18.04.

## Download the Latest Binary Release
```
wget https://github.com/tellor-io/TellorMiner/releases/latest/download/TellorMiner
```

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
To utilize your GPU, you need to add the following line to your `config.json` file:
 
         "useGPU":true,


## Download the PSR File
The `psr.json` file contains information about the different request and associated URLs. Download the latest version:
```
wget https://raw.githubusercontent.com/tellor-io/TellorMiner/master/psr.json
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
## Updating TellorMiner
To update TellorMiner, just download the latest binary release
```
wget https://github.com/tellor-io/TellorMiner/releases/latest/download/TellorMiner
```
And get the latest PSR file:
```
wget https://raw.githubusercontent.com/tellor-io/TellorMiner/master/psr.json
```

### DISCLAIMER


    Mine at your own risk.  

    Mining requires you deposit 1000 Tellor Tributes.  These are a security deposity.  If you are a malicious actor (aka submit a bad value), the community can vote to slash your 1000 tokens.  

    Mining also requires submitting on-chain transactions on Ethereum.  These transactions cost gas (ETH) and can sometimes be signifiant if the cost of gas on EThereum is high (i.e. the network is clogged).  Please reach out to the community to find the best tips for keeping gas costs under control or at least being aware of the costs. 

    If you are building a competing client, please contact us.  A lot of the miner specifications are off-chain and a significant portion of the mining process hinges on the consensus of the Tellor community to determine what proper values are.  Competing clients that change different pieces run the risk of being disputed by the commmunity.  

    There is no guaruntee of profit from mining. 

    There is no promise that Tellor Tributes currently hold or will ever hold any value. 