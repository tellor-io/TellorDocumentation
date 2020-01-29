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
At this point, you will be able to run the miner use go if you `cd TellorMiner` and run commands from inside of the `TellorMiner` directory.

### DISCLAIMER


    Mine at your own risk.  

    Mining requires you deposit 1000 Tellor Tributes.  These are a security deposity.  If you are a malicious actor (aka submit a bad value), the community can vote to slash your 1000 tokens.  

    Mining also requires submitting on-chain transactions on Ethereum.  These transactions cost gas (ETH) and can sometimes be signifiant if the cost of gas on EThereum is high (i.e. the network is clogged).  Please reach out to the community to find the best tips for keeping gas costs under control or at least being aware of the costs. 

    If you are building a competing client, please contact us.  A lot of the miner specifications are off-chain and a significant portion of the mining process hinges on the consensus of the Tellor community to determine what proper values are.  Competing clients that change different pieces run the risk of being disputed by the commmunity.  

    There is no guaruntee of profit from mining. 

    There is no promise that Tellor Tributes currently hold or will ever hold any value. 