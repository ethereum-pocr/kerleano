# Kerleano : Get Started

Kerleano is the name of the test network of the Proof of Climate awaReness consensus.

The following page will guide you to launch your node, first as a client node (synchronizing and reading the chain and to submit transaction) then if desired transform it into a sealer node (producing blocks). 

It is possible to run a client node quickly via a docker image. See [Quick launch](./quick-start-with-docker.md).

## Disclamers
The authors of this guide takes no responsibility nor liabilities of the result of the execution of the guide and consequences to the user environment. You are expected to understand the actions your are taking and the consequences of these actions.

## Pre requisites
* Being familiar with the Ethereum ecosystem
* Be familiar with linux command line (if you target windows, consult the guide [Get Started for windows](./get-started-for-windows.md) )
* Have a computer/VM connected to internet with at least 2Go of free memory
  * Architecture can be anything (if other than `amd64` you may need to recompile the golang binary)
  * OS can be anything (but if different from linux or darwin (Mac Os) you may need to recompile the golang binary)
  * Being administrator on the machine is not mandatory
* Have an editor (e.g. vi)
* Have a download tool (e.g. curl)
  


## 0. Prepare the environment
We assume that the target machine has a user (here we name it `kerleano`) 
* Create a folder to host the environment: `mkdir ~/kerleano`
  

## 1. Get `geth` binary
`geth` is the binary of the Ethereum client implemented in golang.   
For the Proof of Climate awaReness consensus, the software sources and binary resides in https://github.com/ethereum-pocr/go-ethereum repository.

### 1.1 Download from the repository (amd64 only)

```sh
cd ~/kerleano
mkdir bin
curl -o ./bin/geth -L https://github.com/ethereum-pocr/go-ethereum/releases/download/v1.10.26-pocr-2.0.0/geth
sha256sum bin/geth # should display 472e357bf9914fcfa6618716985cb49318586e905808b84dd847166f0ae0034a
chmod +x bin/geth # to make the binary executable
bin/geth version # to check it is functional in your environment
```
Output of the version command:
```
Geth
Version: 1.10.26-pocr-2.0.0
Git Commit: 18921cf46b541e635921f5eb0c4b4eaaf9c8c38b
Architecture: amd64
Go Version: go1.19.4
Operating System: linux
GOPATH=
GOROOT=
```

### 1.2 Build from source

If you haven't been able to make the binary run or that you are not in an `amd64` architecture, you need to recompile the code. Else skip this section.

#### 1.2.1 Golang compilation environment
* Install golang 1.19.4 or above : `apt install golang`
* Install `make` utility to run makefile : `apt install make`
* Install `git` utility to download git repo : `apt install git`
  
#### 1.2.2 Build the code
* Download the source code (no git history needed)
```sh
cd ~/kerleano
git clone --depth=1 https://github.com/ethereum-pocr/go-ethereum.git 
```
* Actual build
```sh
cd ~/kerleano/go-ethereum
make geth
```
output:
```
env GO111MODULE=on go run build/ci.go install ./cmd/geth
go: downloading github.com/cespare/cp v0.1.0
go: downloading github.com/Azure/azure-sdk-for-go/sdk/storage/azblob v0.3.0
go: downloading golang.org/x/crypto v0.0.0-20210921155107-089bfa567519
....
github.com/ethereum/go-ethereum/console
github.com/ethereum/go-ethereum/cmd/geth
Done building.
Run "./build/bin/geth" to launch geth.
```
#### 1.2.3 Move the code to the bin execution folder

```sh
cd ~/kerleano
mkdir -p bin # may already exists if created before
cp go-ethereum/build/bin/geth bin/geth
```

### 1.3 Make the binary available in PATH

For convenience add the `~/kerleano/bin` folder in the `PATH`
```sh
export PATH=$PATH:~/kerleano/bin
# Add the above line in your shell startup script for convenience
which geth # should display the ~/kerleano/bin location
```


## 2. Initialize the network data
The client node needs 1) to initialize with a valid genesis block and 2) know a few initial live nodes to initialize the peer-to-peers connections

### 2.1 Get the genesis file
The Kerleano genesis file is stored in the [ethereum-pocr.github.io](https://github.com/ethereum-pocr/ethereum-pocr.github.io) repository where the script to produce it are located.

```sh
cd ~/kerleano
curl -o genesis.json -L https://github.com/ethereum-pocr/ethereum-pocr.github.io/releases/download/v2.0.0/kerleano.json
```
If you take the time to look into the file you will see that 
* the chainId is `1804`. network id will be the same
* block period is 4 seconds (ie one block every 4 seconds)
* for address `0x0000000000000000000000000000000000000100` the code of the governance smart contract that resides at this address.

### 2.2 Get the boot nodes
The list of live boot nodes are maintained in the [kerleano](https://github.com/ethereum-pocr/kerleano) repository.

```sh
cd ~/kerleano
curl -o bootnodes -L https://raw.githubusercontent.com/ethereum-pocr/kerleano/main/BOOTNODES
```

This list can change from time to time, so it may be wise to apply regular update of that file.

### 2.3 Initialize the network database
`geth` stores its database and other files in folders designed as `datadir` and it needs to be initialized with the genesis file

```sh
cd ~/kerleano
geth init --datadir .ethereum ./genesis.json
```
Expected output:
```javascript
INFO [05-05|16:21:44.875] Maximum peer count                       ETH=50 LES=0 total=50
INFO [05-05|16:21:44.877] Smartcard socket not found, disabling    err="stat /run/pcscd/pcscd.comm: no such file or directory"
INFO [05-05|16:21:44.880] Set global gas cap                       cap=50,000,000
INFO [05-05|16:21:44.882] Allocated cache and file handles         database=/home/kerleano/kerleano/.ethereum/geth/chaindata cache=16.00MiB handles=16
INFO [05-05|16:21:44.906] Opened ancient database                  database=/home/kerleano/kerleano/.ethereum/geth/chaindata/ancient/chain readonly=false
INFO [05-05|16:21:44.907] Writing custom genesis block 
INFO [05-05|16:21:44.910] Persisted trie from memory database      nodes=13 size=1.80KiB time="380.229µs" gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [05-05|16:21:44.913] Successfully wrote genesis state         database=chaindata hash=23dafd..0f9e5d
INFO [05-05|16:21:44.913] Allocated cache and file handles         database=/home/kerleano/kerleano/.ethereum/geth/lightchaindata cache=16.00MiB handles=16
INFO [05-05|16:21:44.938] Opened ancient database                  database=/home/kerleano/kerleano/.ethereum/geth/lightchaindata/ancient/chain readonly=false
INFO [05-05|16:21:44.938] Writing custom genesis block 
INFO [05-05|16:21:44.940] Persisted trie from memory database      nodes=13 size=1.80KiB time="96.447µs"  gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [05-05|16:21:44.941] Successfully wrote genesis state         database=lightchaindata hash=23dafd..0f9e5d
```

## 3. Start the node as a client node

We will first start the node in pure command line then create the startup script for convenience
### 3.1 Manual start
Note that the bash shell or compatible for the `readarray` command
```bash
cd ~/kerleano
BOOTNODES=$(readarray -t ARRAY < bootnodes; IFS=,; echo "${ARRAY[*]}") # convert the file into a comma separated list
NETWORK_ID=$(grep chainId genesis.json | grep -Eo '[0-9]+') # get the value of the chain Id and use it as network id as well
PUBLIC_IP=$(curl -s ifconfig.me/ip) # needed to expose your node to other, else you will not be reachable in discovery
SYNCMODE=snap # tells to synchronize without validating the transactions (faster but imply you trust the bootnodes)

geth --networkid $NETWORK_ID --datadir .ethereum --bootnodes "$BOOTNODES" --syncmode $SYNCMODE --http --http.addr=0.0.0.0 --http.port=8545 --http.api=web3,eth,net,clique --http.corsdomain=* --http.vhosts=* --ws --ws.addr=0.0.0.0 --ws.port=8546 --ws.api=web3,eth,net,clique --ws.origins=* --nat extip:$PUBLIC_IP
```
Output: __it will take several minutes to synchronize up to the top of the chain__

It took 9 minutes for the 3,056,729 blocks. The database size is 1 379Mo = 1.4Go. This will obviously depends on the moment when you synchronize and the bandwidth of your system.

```javascript
INFO [05-05|16:45:10.306] Maximum peer count                       ETH=50 LES=0 total=50
INFO [05-05|16:45:10.307] Smartcard socket not found, disabling    err="stat /run/pcscd/pcscd.comm: no such file or directory"
INFO [05-05|16:45:10.311] Set global gas cap                       cap=50,000,000
INFO [05-05|16:45:10.313] Allocated trie memory caches             clean=154.00MiB dirty=256.00MiB
INFO [05-05|16:45:10.313] Allocated cache and file handles         database=/home/kerleano/kerleano/.ethereum/geth/chaindata cache=512.00MiB handles=524,288
INFO [05-05|16:45:10.346] Opened ancient database                  database=/home/kerleano/kerleano/.ethereum/geth/chaindata/ancient/chain readonly=false
INFO [05-05|16:45:10.347]  
INFO [05-05|16:45:10.347] --------------------------------------------------------------------------------------------------------------------------------------------------------- 
INFO [05-05|16:45:10.347] Chain ID:  1804 (unknown) 
INFO [05-05|16:45:10.347] Consensus: PoCR (proof-of-climate-awareness) 
INFO [05-05|16:45:10.347]  
INFO [05-05|16:45:10.347] Pre-Merge hard forks: 
INFO [05-05|16:45:10.347]  - Homestead:                   0        (https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/homestead.md) 
INFO [05-05|16:45:10.348]  - Tangerine Whistle (EIP 150): 0        (https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/tangerine-whistle.md) 
INFO [05-05|16:45:10.348]  - Spurious Dragon/1 (EIP 155): 0        (https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/spurious-dragon.md) 
INFO [05-05|16:45:10.348]  - Spurious Dragon/2 (EIP 158): 0        (https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/spurious-dragon.md) 
INFO [05-05|16:45:10.348]  - Byzantium:                   0        (https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/byzantium.md) 
INFO [05-05|16:45:10.348]  - Constantinople:              0        (https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/constantinople.md) 
INFO [05-05|16:45:10.348]  - Petersburg:                  0        (https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/petersburg.md) 
INFO [05-05|16:45:10.348]  - Istanbul:                    0        (https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/istanbul.md) 
INFO [05-05|16:45:10.348]  - Berlin:                      0        (https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/berlin.md) 
INFO [05-05|16:45:10.348]  - London:                      0        (https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/london.md) 
INFO [05-05|16:45:10.348]  
INFO [05-05|16:45:10.348] The Merge is not yet available for this network! 
INFO [05-05|16:45:10.348]  - Hard-fork specification: https://github.com/ethereum/execution-specs/blob/master/network-upgrades/mainnet-upgrades/paris.md 
INFO [05-05|16:45:10.348] --------------------------------------------------------------------------------------------------------------------------------------------------------- 
INFO [05-05|16:45:10.348]  
INFO [05-05|16:45:10.349] Initialising Ethereum protocol           network=1804 dbversion=<nil>
INFO [05-05|16:45:10.350] Loaded most recent local header          number=0 hash=23dafd..0f9e5d td=1 age=49y6mo3w
INFO [05-05|16:45:10.350] Loaded most recent local full block      number=0 hash=23dafd..0f9e5d td=1 age=49y6mo3w
INFO [05-05|16:45:10.350] Loaded most recent local fast block      number=0 hash=23dafd..0f9e5d td=1 age=49y6mo3w
WARN [05-05|16:45:10.350] Failed to load snapshot, regenerating    err="missing or corrupted snapshot"
INFO [05-05|16:45:10.350] Rebuilding state snapshot 
INFO [05-05|16:45:10.351] Resuming state snapshot generation       root=91f00f..82d27c accounts=0 slots=0 storage=0.00B dangling=0 elapsed="569.077µs"
INFO [05-05|16:45:10.352] Regenerated local transaction journal    transactions=0 accounts=0
INFO [05-05|16:45:10.352] Generated state snapshot                 accounts=9 slots=0 storage=416.00B dangling=0 elapsed=1.829ms
INFO [05-05|16:45:10.353] Gasprice oracle is ignoring threshold set threshold=2
INFO [05-05|16:45:10.353] Stored checkpoint snapshot to disk       number=0 hash=23dafd..0f9e5d
WARN [05-05|16:45:10.353] Error reading unclean shutdown markers   error="leveldb: not found"
WARN [05-05|16:45:10.353] Engine API enabled                       protocol=eth
WARN [05-05|16:45:10.354] Engine API started but chain not configured for merge yet 
INFO [05-05|16:45:10.354] Starting peer-to-peer node               instance=Geth/v1.10.26-pocr-2.0.0-d1d51931-20230303/linux-amd64/go1.18.1
INFO [05-05|16:45:10.367] Mapped network port                      proto=tcp extport=30303 intport=30303 interface=ExtIP(78.203.4.230)
INFO [05-05|16:45:10.367] Mapped network port                      proto=udp extport=30303 intport=30303 interface=ExtIP(78.203.4.230)
INFO [05-05|16:45:10.369] New local node record                    seq=1,683,305,110,364 id=5dcd21edde7c4124 ip=78.203.4.230 udp=30303 tcp=30303
INFO [05-05|16:45:10.373] Started P2P networking                   self=enode://1478dfe0a1711b4aa4a946f68b1f0b1d438df7004ccf69bdcfa4aefb3242c48f5a7a2d0098f8b9db16e6eea4d7ee3a87b9a40af4c2a0504d6e60c0dc5a08c2f2@78.203.4.230:30303
INFO [05-05|16:45:10.379] IPC endpoint opened                      url=/home/kerleano/kerleano/.ethereum/geth.ipc
INFO [05-05|16:45:10.380] Generated JWT secret                     path=/home/kerleano/kerleano/.ethereum/geth/jwtsecret
INFO [05-05|16:45:10.381] HTTP server started                      endpoint=[::]:8545 auth=false prefix= cors=* vhosts=*
INFO [05-05|16:45:10.382] WebSocket enabled                        url=ws://[::]:8546
INFO [05-05|16:45:10.383] WebSocket enabled                        url=ws://127.0.0.1:8551
INFO [05-05|16:45:10.384] HTTP server started                      endpoint=127.0.0.1:8551 auth=true  prefix= cors=localhost vhosts=localhost
INFO [05-05|16:45:20.364] Block synchronisation started 
INFO [05-05|16:45:21.567] Imported new block headers               count=192 elapsed=28.406ms    number=192 hash=62a329..6bdc19 age=4mo3w1d
INFO [05-05|16:45:21.568] Downloader queue stats                   receiptTasks=0 blockTasks=0 itemSize=650.00B throttle=8192
INFO [05-05|16:45:21.570] Wrote genesis to ancients 
INFO [05-05|16:45:21.580] Imported new block receipts              count=192 elapsed=10.902ms    number=192 hash=62a329..6bdc19 age=4mo3w1d  size=65.50KiB
INFO [05-05|16:45:21.599] Imported new block headers               count=192 elapsed=31.812ms    number=384 hash=6857aa..cb15da age=4mo3w1d
INFO [05-05|16:45:21.615] Imported new block receipts              count=192 elapsed=13.440ms    number=384 hash=6857aa..cb15da age=4mo3w1d  size=64.36KiB
INFO [05-05|16:45:21.654] Imported new block headers               count=192 elapsed=54.439ms    number=576 hash=5f4223..09d2e4 age=4mo3w1d

....

INFO [05-05|16:54:01.938] 💵 Sealer earnings                        block=3,056,728 node=0xCca833ec5Bb4FCeb5145fa2fD65c94c446028Ff6 rank=1.0000 blockReward=2638332094109262869 feeAdjustment=0 burnt=0
INFO [05-05|16:54:01.946] 💵 Sealer earnings                        block=3,056,729 node=0xeC3F218D9876BFC01E9A4601575Fe60F1b2C8F76 rank=0.8100 blockReward=2137048591770621953 feeAdjustment=0 burnt=0
INFO [05-05|16:54:01.948] Imported new chain segment               blocks=67 txs=0 mgas=0.000 elapsed=230.253ms   mgasps=0.000 number=3,056,729 hash=1db848..a51f71 dirty=191.10KiB
INFO [05-05|16:54:01.951] Upgrading chain index                    type=bloombits percentage=0
INFO [05-05|16:54:02.264] Unindexed transactions                   blocks=90133 txs=71 tail=706,730 elapsed=312.997ms
INFO [05-05|16:54:04.608] Imported new block headers               count=2    elapsed="672.804µs" number=3,056,731 hash=67b6f2..9a1f09
INFO [05-05|16:54:04.609] 💵 Sealer earnings                        block=3,056,730 node=0x1311aEF86D1DB33Db945fc488eEFF1C6105b9593 rank=0.9000 blockReward=2374498071288730643 feeAdjustment=0 burnt=0
INFO [05-05|16:54:04.610] 💵 Sealer earnings                        block=3,056,731 node=0x20C8f6D7E11C42C83C9af2f6144c963315106Fc9 rank=0.6561 blockReward=1731008799119890544 feeAdjustment=0 burnt=0
INFO [05-05|16:54:04.611] Imported new chain segment               blocks=2     txs=0  mgas=0.000 elapsed=3.034ms     mgasps=0.000 number=3,056,731 hash=67b6f2..9a1f09 dirty=196.82KiB
INFO [05-05|16:54:04.612] Unindexed transactions                   blocks=2     txs=0  tail=706,732 elapsed="154.396µs"
INFO [05-05|16:54:04.649] Snap sync complete, auto disabling 
INFO [05-05|16:54:05.119] 💵 Sealer earnings                        block=3,056,732 node=0xC16c63Da5F6e464148Ffc0Bcef9e761A16Af6004 rank=0.7290 blockReward=1923342871305117034 feeAdjustment=0 burnt=0
INFO [05-05|16:54:05.120] Imported new chain segment               blocks=1     txs=0  mgas=0.000 elapsed=1.619ms     mgasps=0.000 number=3,056,732 hash=8681a8..ad2b98 dirty=199.69KiB
INFO [05-05|16:54:05.120] Unindexed transactions                   blocks=1     txs=0  tail=706,733 elapsed="282.17µs"
INFO [05-05|16:54:09.562] 💵 Sealer earnings                        block=3,056,733 node=0xCca833ec5Bb4FCeb5145fa2fD65c94c446028Ff6 rank=1.0000 blockReward=2638330049301283676 feeAdjustment=0 burnt=0
INFO [05-05|16:54:09.563] Imported new chain segment               blocks=1     txs=0  mgas=0.000 elapsed=1.700ms     mgasps=0.000 number=3,056,733 hash=241126..6e56ca dirty=202.73KiB
```

Press CTRL+C to stop the process.    
Restart the process and the synchronization will resume where you left it.

### 3.2 Create a startup script 

Add the following in `start_node.sh`, then apply executable right to it `chmod +x start_node.sh`.  

You can obviously adjust the port and config as needed.

```sh
#!/bin/bash

BOOTNODES=$(readarray -t ARRAY < bootnodes; IFS=,; echo "${ARRAY[*]}") # convert the file into a comma separated list
NETWORK_ID=$(grep chainId genesis.json | grep -Eo '[0-9]+') # get the value of the chain Id and use it as network id as well
PUBLIC_IP=$(curl -s ifconfig.me/ip) # needed to expose your node to other, else you will not be reachable in discovery
SYNCMODE=snap # tells to synchronize without validating the transactions (faster but imply you trust the bootnodes)

nohup geth --networkid $NETWORK_ID --datadir .ethereum --bootnodes "$BOOTNODES" --syncmode $SYNCMODE \
      --http --http.addr=0.0.0.0 --http.port=8545 --http.api=web3,eth,net,clique --http.corsdomain=* --http.vhosts=* \
      --ws --ws.addr=0.0.0.0 --ws.port=8546 --ws.api=web3,eth,net,clique --ws.origins=* \
      --nat extip:$PUBLIC_IP 2>&1 1>> geth.log & 

```

### 3.3 Connect and interact with the node

The client node exposes an http (port 8545) and web socket (port 8546) JSON Rpc endpoint to dialog with the node.

To test the connection try
```sh
geth attach http://localhost:8545
# or
geth attach ws://localhost:8546
```

You can then connect your Metamask (or compatible wallet) to that node.

## 4. Convert into a sealer

A sealer needs to have a private key to sign the blocks. That private key is identified by its public address as `miner.etherbase` (also known as `coinbase`) and it is the wallet that will receive the block rewards and transaction fees.   

A sealer node therefore keeps the wallet on its local drive (unless a remote wallet solution is setup, such as `clef` or hardware wallet), hence we do not want to expose the JSON Rpc endpoint to prevent the risk of anyone signing a transaction on behalf of the node.

### 4.1 Creating the sealer etherbase wallet

Geth provide a tool to generate a private key in the [Web3 Secret Storage format](https://ethereum.org/en/developers/docs/data-structures-and-encoding/web3-secret-storage/) :

The command will generate a file in the `.keystore` folder
```sh
geth account new --keystore .keystore 
```
output:

```
Your new account is locked with a password. Please give a password. Do not forget this password.
Password: 
Repeat password: 

Your new key was generated

Public address of the key:   0x64d0eFdeFb40E39A5ea946A0E132Dc0b690BD2c1
Path of the secret key file: /home/kerleano/kerleano/.keystore/UTC--2023-05-05T17-51-10.585595624Z--64d0efdefb40e39a5ea946a0e132dc0b690bd2c1

- You can share your public address with anyone. Others need it to interact with you.
- You must NEVER share the secret key with anyone! The key controls access to your funds!
- You must BACKUP your key file! Without the key, it's impossible to access account funds!
- You must REMEMBER your password! Without the password, it's impossible to decrypt the key!
```

Save the provided password into a text file so it can be used in script later: `echo "your password" > .keystore/password` 

### 4.2 Modify the script to act as a sealer

```sh
#!/bin/bash

BOOTNODES=$(readarray -t ARRAY < bootnodes; IFS=,; echo "${ARRAY[*]}") # convert the file into a comma separated list
NETWORK_ID=$(grep chainId genesis.json | grep -Eo '[0-9]+') # get the value of the chain Id and use it as network id as well
PUBLIC_IP=$(curl -s ifconfig.me/ip) # needed to expose your node to other, else you will not be reachable in discovery
SYNCMODE=full # prefer a full transaction verification on sealer
ETHERBASE=0x64d0eFdeFb40E39A5ea946A0E132Dc0b690BD2c1 # replace with the locally created wallet

nohup geth --networkid $NETWORK_ID --datadir .ethereum --bootnodes "$BOOTNODES" --syncmode $SYNCMODE \
      --keystore .keystore --miner.etherbase $ETHERBASE --unlock $ETHERBASE --password .keystore/password \
      --miner.extradata="Your name as sealer (32b max)" \
      --mine --miner.gasprice 1000000000 \
      --nat extip:$PUBLIC_IP 2>&1 1>> geth.log & 
```

After restarting, log will show that you are unable to sign blocks 
```
WARN [05-05|18:41:13.145] Block sealing failed                     err="unauthorized signer"
```

This is because you have not been included as a sealer in the clique consensus (the base consensus of the PoCR)

To connect to your node, you can only do it locally using the IPC protocol via
```sh
geth attach .ethereum/geth.ipc 
```

### 4.3 Request that your node participate to the Consensus

To request the community that your sealer be added, you must [submit an issue](https://github.com/ethereum-pocr/kerleano/issues) with your sealer address (the etherbase above). You must state your identity and your objectives so existing participants in the Kerleano test network can assess if they should allow you or not.

Once approved by a majority of the sealers (N/2+1 where N is the number of existing sealers) your node will start creating blocks.     
Note that for the production network the onboarding process is more restrictive and is detailled in the whitepaper.

Existing sealers will connect on their node and run the following command to accept your node:
```sh
geth attach .ethereum/geth.ipc 
> clique.propose("0x64d0eFdeFb40E39A5ea946A0E132Dc0b690BD2c1", true)
```

### 4.4 Audit your node to get an environmental footprint (in testnet)

Until your node has been evaluated for its environmental footprint by an independant certified auditor, your sealer will not earn any CRC despite the fact you are creating blocks and processing transactions.

So you must reach to a declared auditor. Anyone can become an auditor but it needs to pledge a certain quantity of CRC to be able to audit.

To perform this audit, the auditor goes to the Governance site https://ethereum-pocr.github.io/ with metamask (or operate directly by calling the [Governance smart contracts](https://github.com/ethereum-pocr/ethereum-pocr.github.io/blob/main/sc-carbon-footprint/src/Governance.sol) )

You can monitor the status of your node at the Governance site in the publicly available dashboard (requires metamask).
