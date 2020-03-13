# Run TellorMiner and Join the Pool
These instructions are for running the official TellorMiner as part of a Tellor pool and without staking 1000 TRB. 

## Download the TellorMiner
First, visit GitHub and download the latest release of [TellorMiner](https://github.com/tellor-io/TellorMiner/releases).

Download the Linux or Windows executable and place it into a directory on your computer.

## Create a Configuration File
Next, you will need to create a `config.json` file in the same directory as the TellorMiner executable.

Start by copying this [config.json](https://github.com/tellor-io/TellorMiner/blob/master/config.json) file that is checked into the TellorMiner GitHub repository.

### Edit the Configuration File
Change the following configuration options:
1. `nodeURL`: You will need to get an Project ID from [Infura](https://infura.io/) or replace this with another Ethereum node that supports RPC connections.
2. `privateKey`: Replace this with your Ethereum account private key
3. `publicAddress` and `serverWhitelist`: Replace these with your Ethereum account public address without the 0x prefix
4. Add the following options to tell the miner to connect to the pool:
```
"enablePoolWorker": true,
"poolURL": "POOL_URL"
```
Where the `POOL_URL` is the link to your pool. Current Tellor mining pools:
* Coming soon

5. If you're using GPU, add the following:
```
"gpuConfig":{
  "default":{
    "groupSize":256,
    "groups":4096,
    "count":16
  }
}
```

## Create a Logging Configuration File
Next, you will need to create a `loggingConfig.json` file in the same directory as the TellorMiner executable.

Start by copying this [loggingConfig.json](https://github.com/tellor-io/TellorMiner/blob/master/loggingConfig.json) file that is checked into the TellorMiner GitHub repository.

## Run the TellorMiner
Now run the miner, on Linux:
```
.\TellorMiner -miner
```
and on Windows:
```
TellorMiner.exe -miner
```


### DISCLAIMER


    Mine at your own risk.  

    Mining requires you deposit 1000 Tellor Tributes.  These are a security deposity.  If you are a malicious actor (aka submit a bad value), the community can vote to slash your 1000 tokens.  

    Mining also requires submitting on-chain transactions on Ethereum.  These transactions cost gas (ETH) and can sometimes be signifiant if the cost of gas on EThereum is high (i.e. the network is clogged).  Please reach out to the community to find the best tips for keeping gas costs under control or at least being aware of the costs.

    If you are building a competing client, please contact us.  A lot of the miner specifications are off-chain and a significant portion of the mining process hinges on the consensus of the Tellor community to determine what proper values are.  Competing clients that change different pieces run the risk of being disputed by the commmunity.  

    There is no guaruntee of profit from mining.

    There is no promise that Tellor Tributes currently hold or will ever hold any value.
