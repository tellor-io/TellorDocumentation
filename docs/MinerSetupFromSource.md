# Run TellorMiner from Source
These instructions are for installing and running TellorMiner from source on Linux. These have been tested on Ubuntu 18.04.

## Setup TellorMiner
### Install Build Essentials
```
apt-get install build-essential
```
### Install Go
TellorMiner uses go so start by installing go:
```
wget https://dl.google.com/go/go1.13.1.linux-amd64.tar.gz
tar -xvf go1.13.1.linux-amd64.tar.gz
mv go /usr/local
cat <<EOT >> ~/.profile
export GOPATH=$HOME/go
export PATH=$PATH:/usr/local/go/bin:$GOPATH/bin
EOT
source ~/.profile
```

### Setup Go Project Directory Structure
You will need to create the following directory structure in the location where you wish to install TellorMiner.
```
---./bin/
---./pkg/
---./src/
   |
   ---./github.com/
      |
      ---./tellor-io/
```
These instructions assume you're installing in your `$HOME` directory and will create this in `$HOME/go`:
```
mkdir go
cd go
mkdir -p src/github.com/tellor-io
mkdir pkg
mkdir bin
```

### Use Git to Download TellorMiner
Navigate into `src/github.com/tellor-io/` and clone the TellorMiner:
```
cd ~/go/src/github.com/tellor-io
git clone https://github.com/tellor-io/TellorMiner
```

### Get Go Dependancies for Tellor Miner
Remaining in `src/github.com/tellor-io/`, use `go get` to download and install the dependancies you will need to run TellorMiner. This can take a while to run so be patient:
```
go get -d ./TellorMiner
cd TellorMiner
```

Now you need to generate the opencl files for your build by creating a 'kernelSource.go' file in your pow folder.  To do this:

```
cd pow
go generate #should create a kernelSource.go file
cd ..

# ready to 'go run'  or 'go build'
```
At this point, you will be able to run the miner use go if you `cd TellorMiner` and run commands from inside of the `TellorMiner` directory. A few example commands you can run:
```
# Deposit Initial Stake
go run ./main.go -deposit -config=./config.json -psrPath=./psr.json -logConfig=./loggingConfig.json

# Transfer Tributes
go run ./main.go -transfer -to=<0x...toaddress....> -amount=<number of tributes> -config=./config.json -psrPath=./psr.json -logConfig=./loggingConfig.json
```

## Start Mining
### Update `config.json`
To run TellorMiner you first need to change `config.json`. Do the following changes
1. Set `nodeUrl` to an Ethereum node endpoint (e.g. Infura API endpoint)
2. Set `privateKey` to the private key for the Ethereum wallet you plan to use (note no "0x" prefix)
3. Set `publicAddress` to the public key for the Ethereum wallet you plan to use (note no "0x" prefix)
4. Set `serverWhitelist` so that it includes your public key (this whitelists your miner to use your local dataserver)
5. Add `numProcessors` to the number of processors your computer has
6. Add `requestData` and set the value to `0` so you're not constantly requesting data from Tellor

### Utilizing your GPU
To utilize your GPU, you need to add the following line to your `config.json` file:
 
    "gpuConfig":{
        "foo":{
                "groupSize":64,
                "groups":128,
                "count":256
        }
    },


### Deposit your initial stake
To deposit your stake initially (assuming you have 1000 Tributes) you can run the following command from inside `~/go/src/github.com/tellor-io/TellorMiner`
```
go run ./main.go -deposit -config=./config.json -psrPath=./psr.json -logConfig=./loggingConfig.json
```

### Run the Data Server
The data server will fetch data from the internet and check the blockchain for miners. You need to run at least 1 data server. It is possible to run the data server and a miner on the same computer. You will start the data server using the following command:
```
go run ./main.go -dataServer -config=./config.json -psrPath=./psr.json -logConfig=./loggingConfig.json
```
After starting the data server, observe the logs it outputs to confirm it's working correctly.

### Run the Miner
Once the data server is running, start the miner by running this command in another terminal or process:
```
go run ./main.go -miner -config=./config.json -psrPath=./psr.json -logConfig=./loggingConfig.json
```
After starting the miner, observe the logs it outputs to confirm it's working correctly.

### DISCLAIMER


    Mine at your own risk.  

    Mining requires you deposit 1000 Tellor Tributes.  These are a security deposity.  If you are a malicious actor (aka submit a bad value), the community can vote to slash your 1000 tokens.  

    Mining also requires submitting on-chain transactions on Ethereum.  These transactions cost gas (ETH) and can sometimes be signifiant if the cost of gas on EThereum is high (i.e. the network is clogged).  Please reach out to the community to find the best tips for keeping gas costs under control or at least being aware of the costs. 

    If you are building a competing client, please contact us.  A lot of the miner specifications are off-chain and a significant portion of the mining process hinges on the consensus of the Tellor community to determine what proper values are.  Competing clients that change different pieces run the risk of being disputed by the commmunity.  

    There is no guaruntee of profit from mining. 

    There is no promise that Tellor Tributes currently hold or will ever hold any value. 