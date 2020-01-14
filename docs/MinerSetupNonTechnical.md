# Tellor Miner

This is the workhorse of the Miner system as it takes on solving the PoW challenge.  

It's built on Go and utilizes a split structure.  The database piece is a LevelDB that keeps track of all variables (challenges, difficulty, values to submit, etc.) and the miner simply solves the PoW challenge.  This enables parties to split the pieces for optimization.

<p align="center">
<img src="./img/minerspecs.png" width="450" height="400" alt="minerspecs">
</p>


### Tellor Deployed Addresses

Mainnet - [0x0ba45a8b5d5575935b8158a88c631e9f9c95a2e5](https://etherscan.io/address/0x0ba45a8b5d5575935b8158a88c631e9f9c95a2e5)

Rinkeby - [0x724D1B69a7Ba352F11D73fDBdEB7fF869cB22E19](https://rinkeby.etherscan.io/address/0x724D1B69a7Ba352F11D73fDBdEB7fF869cB22E19)

</br>


The following is documentation on launching the miner from a new AWS Linux box.  It was written for the non-technical audience

## Table of Contents
* [How to launch the miner](#launch)
* [How to update the miner](#update)

## How to launch the miner <a name="launch"> </a>  

This instructions describe how to install the miner to a clean AWS Ubuntu/Linux box that is at minimum a t2.small (or you will run into data limitations).

You will be creating 7 files using the nano command and copying and pasting the code included below for each file. 


* Open a terminal connected to your AWS instance and run the following commands:

```bash
sudo apt-get update

wget https://github.com/tellor-io/TellorMiner/releases/latest/download/TellorMiner 

nano config.json

```
* Update the following three variables within the config.json file below:

node_url: (enter your own infura / node address)
publicAddress: (note no “0x” prefix)
privateKey: (note no “0x” prefix)

config.json

```json
{
    "contractAddress": "0x0Ba45A8b5d5575935B8158a88C631E9F9C95a2e5",
    "nodeURL": "https://mainnet.infura.io/v3/7bbbbbbbbbbbbbbb",
    "privateKey": "4bdc16637633fa4b4854670fbb83fa254756798009f5bbbbbbbbbbbbbbbbbbbbbb",
    "databaseURL":"http://localhost7545",
    "publicAddress": "e037ec8ec9ec4238bbbbbbbbbbbbbbbbbbbbbbbbbbbbbb",
    "serverHost": "localhost",
    "serverPort": 5001,
    "ethClientTimeout": 3000,
    "trackerCycle": 22,
    "requestData":12,
    "gpuConfig":{
        "foo":{
                "groupSize":64,
                "groups":128,
                "count":256
        }
    },
 "trackers": [
        "balance",
        "currentVariables",
        "disputeStatus",
        "gas",
        "top50",
        "tributeBalance",
        "fetchData",
        "psr"
    ],
    "dbFile": "/tmp/tellor/tellor_db"
}


```
** Note that if you do not want to use the GPU miner, simply delete the "gpuConfig" variable


* Copy and paste the code into the terminal and use the hot keys below to save config.json and the nano command to create loggingConfig.json:

```bash
Ctrl + o
Enter
Ctrl + x

nano loggingConfig.json

```

* Copy and paste the code below on the terminal to loggingConfig.json:

```json
[
    {
        "component": "config.Config",
        "level": "INFO"
    },
    {
        "component":"db.DB",
        "level": "INFO"
    },
    {
        "component": "rpc.client",
        "level": "INFO"
    },
    {
        "component": "rpc.ABICodec",
        "level": "INFO"
    },
    {
        "component": "rpc.mockClient",
        "level": "INFO"
    },
    {
        "component": "tracker.Top50Tracker",
        "level": "INFO"
    },
    {
        "component": "tracker.FetchDataTracker",
        "level": "INFO"
    }
]
```

* Use the hot keys below to save loggingConfig.json and create psr.json:

```bash
wget https://raw.githubusercontent.com/tellor-io/TellorMiner/master/psr.json 

```

* Run the following commands from the terminal to stake your 1000 TRB:

```bash

./TellorMiner -deposit -psrPath=./psr.json -config=./config.json -logConfig=./loggingConfig.json


```
* Run the following commands from the terminal to start the miner:

```bash

./TellorMiner -miner -dataServer -psrPath=./psr.json -config=./config.json -logConfig=./loggingConfig.json


```
## How to update the miner <a name="update"> </a>  

* Open a terminal

* Stop the process and delete the Miner:

```bash
rm TellorMiner
```

* To update Tellor miner run:
```bash
wget https://github.com/tellor-io/TellorMiner/releases/latest/download/TellorMiner 

```


* Once you have made the updates either to the psr.json or the miner, simply restart the miner


### DISCLAIMER


    Mine at your own risk.  

    Mining requires you deposit 1000 Tellor Tributes.  These are a security deposity.  If you are a malicious actor (aka submit a bad value), the community can vote to slash your 1000 tokens.  

    Mining also requires submitting on-chain transactions on Ethereum.  These transactions cost gas (ETH) and can sometimes be signifiant if the cost of gas on EThereum is high (i.e. the network is clogged).  Please reach out to the community to find the best tips for keeping gas costs under control or at least being aware of the costs. 

    If you are building a competing client, please contact us.  A lot of the miner specifications are off-chain and a significant portion of the mining process hinges on the consensus of the Tellor community to determine what proper values are.  Competing clients that change different pieces run the risk of being disputed by the commmunity.  

    There is no guaruntee of profit from mining. 

    There is no promise that Tellor Tributes currently hold or will ever hold any value. 