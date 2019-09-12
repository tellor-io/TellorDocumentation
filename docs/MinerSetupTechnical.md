## How to launch the miner <a name="launch"> </a>  

Grab the latest Binary:

```
wget https://github.com/tellor-io/TellorMiner/releases/latest/download/TellorMiner 

```

Create and update the following three variables within the config.json file:

* node_url: (enter your own infura / node address)
* publicAddress: (note no “0x” prefix)
* privateKey: (note no “0x” prefix)

https://github.com/tellor-io/TellorMiner/blob/master/config.json

* Download the most recent PSR.json file:

https://github.com/tellor-io/TellorMiner/blob/master/psr.json

Now you can run the miner:

`./TellorMiner -miner -dataServer -psrPath=./psr.json -config=./config.json -logConfig=./loggingconfig.json`


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
* nodeURL(required) - node URL (e.g https://mainnet.infura.io/bbbb or https://localhost:8545 if own node)
* privateKey(required) - privateKey for your address (note, no 0x)
* databaseURL(required) - where you are reading from for the server database (if hosted)
* publicAddress(required) - public address for your miner (note, no 0x)
* ethClientTimeout(required) - timeout for making requests from your node
* trackerSleepCycle(required) - how often your database updates
* trackers(required) - which pieces of the database you update
* dbFile(required) - where you want to store your local database (if self-hosting)
* serverHost(required) - location to host server
* serverPort(required) - port to host server
* fetchTimeout- timeout for requesting data from an API
* requestData- Will your miner request Data if challenge is 0.  If yes, then you will addTip() to this number.  Enter a uint number representing request id to be requested (e.g. 2)
* gasMultiplier - Multiplies the submitted gasPrice (e.g. 2 (will double gas costs))
* gasMax - a max for the gas price in gwei (note: this max comes BEFORE the gas multiplier.  So a max gas cost of 10 gwei, can have gas prices up to 20 if gasMultiplier is 2)

```

* To update Tellor miner run:
```bash
wget https://github.com/tellor-io/TellorMiner/releases/latest/download/TellorMiner 

```

