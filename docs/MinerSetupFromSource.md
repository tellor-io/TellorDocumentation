This file is the Setup to build from src. 

For those non-Go programmers, the proper setup(folder structure) for the project is:
<pre>
$HOME/<whatever>
---bin 
---pkg 
---src 
   | 
   ---github.com 
      | 
      ---tellor-io 
         |
         ---TellorMiner
</pre>

## Instructions

Open a terminal and cd into tellor-io
```bash
Set GOPATH to $HOME/<whatever>
Set GOBIN to $HOME/<whatever>/bin 

cd into $HOME/<whatever>/src/github.com/tellor-io/
run "git clone https://github.com/tellor-io/TellorMiner"  // this will create the folder TellorMiner
run "go get -d ./TellorMiner"
```

Now you're ready to build/test


When building/testing, go into src/github.com/tellor-io/TellorMiner and run "go test ./<package-to-test> -config=<path to config>
e.g.  go test -v ./tracker  
or to run
go run C:/company/code/go/src/github.com/tellor-io/TellorMiner/main.go -config=C:/company/code/go/src/github.com/tellor-io/TellorMiner/config.json
and it will test from there.


Now edit the config.json file with your private key, public key, and node_url (the rest are advanced options)

The privateKey, publicKey, and nodeUrl all are dummies.  If you want to run it on localhost, you'll need to deploy a Tellor Contract. Docmumentation on that can be found here:
https://github.com/tellor-io/TellorCore

But if you just want to try it rinkeby, use an infura id and kick it off. Note, you'll need tokens from us to stake as a Miner (on mainnet or Rinkeby). For more info, shoot us email at info@tellor.io or join our discord

For running the file:

./runMain.sh -miner -dataServer

Note it will kick off both the miner and the database

To transfer tributes

go run ./main.go -transfer -to=<0x...toaddress....> -amount=<number of tributes> -config=./config.json -psrPath=./psr2.json -logConfig=./loggingConfig.json

e.g. go run ./main.go -transfer -to=0x2f51c4bf6b66634187214a695be6cdd344d4e9d1 -amount=100 -config=./config.json -psrPath=./psr2.json -logConfig=./loggingConfig.json

To deposit your stake initially (assuming you have 1000 Tributes)

go run ./main.go -deposit -config=./config.json -psrPath=./psr2.json -logConfig=./loggingConfig.json 
