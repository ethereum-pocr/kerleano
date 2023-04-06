## Kerleano

# Network infos

  *  Use **[Issues](https://github.com/ethereum-pocr/kerleano/issues)** to submit a request to joint kerleano network or read the existing requests.   
      Use the Issues list also to follow instruction from the development team in case of reset of the network.   
      **So subscribe to the issues as you need to get notified when a node need your accaptance or when the testnet needs to be reset.**

  *  Use **[Network infos](./nodes-infos.md)** to get `enodes` url, `rpc`and `websocket` endpoints to join and interact with the network

  *  The RPC endpoint and Metamask connection is possible with [Chainlist](https://chainlist.org/?testnets=true&search=kerleano)

# Nodes topology

- **Sealer node**: is an ethereum node that will be authorized to seal blocks and get CRC as rewards once the node is given an environmental footprint by an auditor.     
The sealer node will generate an account (public and private keys) and his public address (aka wallet address) should be authorized by the other sealers of the network (using `clique.propose("address", true)` POA clique consensus).    
The network `kerleano` is initilized with a given address `0x6e45c195e12d7fe5e02059f15d59c2c976a9b730` (the network is already initilized by tech team but you can check the steps of [How kerleano network is initilized here](docs/init_Kerleano_network.md)). 
Once the network is set, this address will be removed (using `clique.propose("address", false)` POA clique consensus).    
Sealer node only exposes to the internet the Whisper (`30303` tcp/udp port) protocol to dialog in encrypted way with other nodes on the network (peers). 

- **Client node**: is a readonly ethereum node that will read all the blocs of the network and expose RPC and Websocket endpoints in order to let apps (monitoring, block explorers, front wallet...) get infos about the network and submit their transactions to the network.


# Join the network

If you want to join the netwok as a client or a sealer node check this doc

  * See [Join kerleano network](docs/join_Kerleano_network.md)



# Check node status after joining the network

Once the configuration of your node(s) to join the network is done, check in your log file that the node start `syncing` with the `kerleano` network, log example:

```code
INFO [04-19|10:59:11.465] Carbon footprint node                    result=00000000000000000000000000000000000000000000000000000000000009c4 node=0x1311aEF86D1DB33Db945fc488eEFF1C6105b9593
INFO [04-19|10:59:11.465] Calculated reward based on footprint     block=231,465 node=0x1311aEF86D1DB33Db945fc488eEFF1C6105b9593 total=7500 nb=2 footprint=2500 reward=1,307,692,307,692,307,692
INFO [04-19|10:59:11.465] Increasing the total crypto              from=132761068132408598043952 to=132762375824716290351644
INFO [04-19|10:59:11.466] Imported new chain segment               blocks=1  txs=0 mgas=0.000 elapsed=1.658ms     mgasps=0.000  number=231,465 hash=4e9bac..f00a51 dirty=357.74KiB

```

You must see this line with `Imported new chain segment` and `number=xxx` corresponding to the block read and imported from the kerleano network

When you start a fresh node with empty datadir, you should see the `Imported new chain segment` log line from block `number=1` until the last block of the network. 


You can check if your node still syncing and the highestBlock of the network using the commands bellow :

```sh
# execute a javascript console to your node
geth attach --datadir $DATADIR 

# check the highestBlock: xxxxx is the last block sealed in the network and currentBlock: xxx is the latest block synced by your node
> eth.syncing
{
  currentBlock: 33337,
  healedBytecodeBytes: 0,
  healedBytecodes: 0,
  healedTrienodeBytes: 0,
  healedTrienodes: 0,
  healingBytecode: 0,
  healingTrienodes: 0,
  highestBlock: 231748,
  startingBlock: 0,
  syncedAccountBytes: 0,
  syncedAccounts: 0,
  syncedBytecodeBytes: 0,
  syncedBytecodes: 0,
  syncedStorage: 0,
  syncedStorageBytes: 0
}

# you can also use eth.blockNumber to see last synched block by the node

> eth.blockNumber
130549

# once the sync is done (currentBlock=highestBlock) the function will return false
> eth.syncing
false

```

you can check the highest block in the [monitoring tool](https://kerleano-monitoring.awnen.com/pocr/?url=wss://kerleano-ws.awnen.com/) 


# Reset the network

As we're still at dev stage, the network may require to be reset, so all the nodes in the network must reset the datadir and init the network with the same `genesis` file `kerleano.json` and `geth` binary used by other nodes

* Delete the `datadir` ($DATADIR defined by the flag `--datadir` when executing geth commands)
```sh
rm -rf $DATADIR
```
* If the genesis `kerleano.json` has changed, download the same version used by other nodes in the network (at this stage we're using `kerleano_version=v1.0`)

```sh
export kerleano_version=v1.0 && \
  curl -f -L -o ~/kerleano.json \
    https://github.com/ethereum-pocr/ethereum-pocr.github.io/releases/download/$kerleano_version/kerleano.json

```

* Init the geth with `kerleano.json`
```sh
geth init --datadir $DATADIR ~/kerleano.json
```

* Start the node again
```sh
# start client node
~/start_client_node.sh

# for sealer node
~/start_sealer_node.sh
```

# Requirements

* For now we only support `os ubuntu arch amd64` for `geth`nodes
* Inbound/Outbound port UDP/TCP `30303` used for discovery by the geth node must be exposed (can be changed when starting `geth` instance with `--port` flag)
* Genesis file `kerleano.json` must be identical to the one used by the nodes in the network
* **Attention!!** The network will stuck if `50%` of the sealers are down, 
To be able to recover the network, those sealers must rejoin the network with same authorized `addresses`, So the  `private keys` and their `passwords` associated to those `addresses` must be stored in a safe place (vault) 
* Make sure to have a `public static ip` address for the nodes you want to share in **[Network infos](https://github.com/ethereum-pocr/kerleano/wiki/Kerleano-network-infos)** and also use the same `nodekey` file (if you've followed this readme `nodekey` file should be in `~/.keystore/nodekey` and passed to `geth` start command with `--nodekey` option) as its used to generate the `enodeId`
`enode` is `enode://${enodeId}@${public_ip_address}:30303`
* For client nodes you can use a reverse proxy of your choice to secure the `websockets` and `rpc` endpoints with `wss` and `https`

Example of nginx config to secure the websocket endpoint :

```code

server {

        listen               443 ssl;
        ssl                  on;
        ssl_certificate      /app/certs/nginx-selfsigned.crt;
        ssl_certificate_key  /app/certs/nginx-selfsigned.key;
        server_name kerleano-ws.awnen.com;


        location / {


                if ($request_method = OPTIONS) {
                        add_header Content-Length 0;
                        add_header Content-Type text/plain;
                        add_header Access-Control-Allow-Origin "*";
                        add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
                        add_header Access-Control-Allow-Headers "Authorization, Content-Type";
                        add_header Access-Control-Allow-Credentials true;
                        return 201;
                }

                proxy_pass http://${IP_OF_THE_NODE}:${WEBSOCKET_PORT}/;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Host $http_host;
        }

}

```

and to secure rpc endpoint :

```code

server {

	listen               443 ssl;
    	ssl                  on;
    	ssl_certificate      /app/certs/nginx-selfsigned.crt;
    	ssl_certificate_key  /app/certs/nginx-selfsigned.key;
    	server_name kerleano-rpc.awnen.com;

	
    	location / {


    		if ($request_method = OPTIONS) {
        		add_header Content-Length 0;
        		add_header Content-Type text/plain;
			add_header Access-Control-Allow-Origin "*";
        		add_header Access-Control-Allow-Methods "GET, POST, PUT, DELETE, OPTIONS";
        		add_header Access-Control-Allow-Headers "Authorization, Content-Type";
        		add_header Access-Control-Allow-Credentials true;
        		return 201;
    		}

		proxy_pass http://${IP_OF_THE_NODE}:${RPC_PORT}/;
		proxy_http_version 1.1;
      		proxy_set_header Upgrade $http_upgrade;
      		proxy_set_header Connection "upgrade";
      		proxy_set_header X-Real-IP $remote_addr;
      		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      		proxy_set_header Host $http_host;
    	}

}

```

default `websocket` port is `8546` and default `rpc` port is `8545`


# Use ansible

You can check [this doc](docs/use_ansible.md) to automate those steps with ansible
