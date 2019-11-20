# Tellor Miner

This is the workhorse of the Miner system as it takes on solving the PoW challenge.  

It's built on Go and utilizes a split structure.  The database piece is a LevelDB that keeps track of all variables (challenges, difficulty, values to submit, etc.) and the miner simply solves the PoW challenge.  This enables parties to split the pieces for optimization.

<p align="center">
<img src="./img/minerspecs.png" width="450" height="400" alt="minerspecs">
</p>


### Tellor Deployed Addresses

Mainnet - [0x0ba45a8b5d5575935b8158a88c631e9f9c95a2e5](https://etherscan.io/address/0x0ba45a8b5d5575935b8158a88c631e9f9c95a2e5)

Rinkeby - [0x3f1571e4dfc9f3a016d90e0c9824c56fd8107a3e](https://rinkeby.etherscan.io/address/0x3f1571e4dfc9f3a016d90e0c9824c56fd8107a3e)

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
    "serverPort": 5000,
    "ethClientTimeout": 3000,
    "trackerCycle": 10,
    "requestData":0,
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
Ctrl + o
Enter
Ctrl + x

nano psr.json

```

* Copy and paste the code below on the terminal to psr.json:

```json
{
    "prespecifiedRequests":[
        {
            "requestID":1,
            "apis":[
                "json(https://api.gdax.com/products/ETH-USD/ticker).price"
            ],
            "transformation" : "value",
            "granularity" : 1000
        }
    ]
}
```

* Use the hot keys below to save psr.json:

```bash
Ctrl + o
Enter
Ctrl + x
```

* Run the following commands from the terminal:

```bash
sudo apt install nodejs 

sudo apt install npm 

sudo npm i -g pm2

./TellorMiner -deposit -psrPath=./psr.json -config=./config.json -logConfig=./loggingConfig.json

nano ecosystem.config.js

```

* Copy and paste the code below on the terminal to ecosystem.config.js:

Note: be sure to check that the path to the runMiner.sh file is correct (e.g. are you home/ubuntu)

```javascript
module.exports = { apps : [ { name: "tellor-miner1", script: "/home/ubuntu/runMiner.sh", exec_mode: "fork", exec_interpreter: "bash"} ] }
```

* Use the hot keys below to save the ecosystem.config.js and create runMiner.sh:

```bash
Ctrl + o
Enter
Ctrl + x

nano runMiner.sh

```

runMiner.sh

```bash
#!/bin/sh
$HOME/TellorMiner -config=$HOME/config.json -miner -dataServer -psrPath=$HOME/psr.json
```

* Use the hot keys below to save the ecosystem.config.js and create ecosystem.config.js:

```bash
Ctrl + o
Enter
Ctrl + x

pm2 start ./ecosystem.config.js

```

## How to update the miner <a name="update"> </a>  

* Open a terminal

* Stop the process by running:

```bash
pm2 stop all

```

* To update PSR run:

```bash
nano PSR.json

```
[paste the new PSR file]

* Use the hot keys below to save psr.json:

```bash
Ctrl + o
Enter
Ctrl + x
```

* To update Tellor miner run:
```bash
wget https://github.com/tellor-io/TellorMiner/releases/latest/download/TellorMiner 

```


* Once you have made the updates either to the psr.json or the miner run:


```bash
pm2 restart all

```

### DISCLAIMER


    Mine at your own risk.  

    Mining requires you deposit 1000 Tellor Tributes.  These are a security deposity.  If you are a malicious actor (aka submit a bad value), the community can vote to slash your 1000 tokens.  

    Mining also requires submitting on-chain transactions on Ethereum.  These transactions cost gas (ETH) and can sometimes be signifiant if the cost of gas on EThereum is high (i.e. the network is clogged).  Please reach out to the community to find the best tips for keeping gas costs under control or at least being aware of the costs. 

    If you are building a competing client, please contact us.  A lot of the miner specifications are off-chain and a significant portion of the mining process hinges on the consensus of the Tellor community to determine what proper values are.  Competing clients that change different pieces run the risk of being disputed by the commmunity.  

    There is no guaruntee of profit from mining. 

    There is no promise that Tellor Tributes currently hold or will ever hold any value. 