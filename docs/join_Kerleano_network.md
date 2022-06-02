# Join the network `kerleano`

We're using 2 `ubuntu` VMs to join the network `kerleano`

**VM1: will be sealer in the network**
- VM1 requirements:
    * os: Ubuntu 20.04.4 LTS
    * ram: 2Gb min
    * cpu: 1 min
    * tcp/udp port 30303: default geth node (can override by `--port` when starting the node)

**VM2 : will be a client node and expose the rpc and ws endpoints of the network**
- VM1 requirements:
    * os: Ubuntu 20.04.4 LTS
    * ram: 2Gb min
    * cpu: 1 min
    * tcp/udp port 30303: default geth port (can override by `--port` when starting the node)
    * port 8545: default rpc port (can override by `--http.port` when starting the node)
    * port 8546: default websocket port (can override by `--ws.port` when starting the node)

# Start sealer node

**1) ssh into the VM**

```sh
ssh user@VM1
```

**2) download geth binary and give exec right**

```sh
# check releases here https://github.com/ethereum-pocr/go-ethereum/releases
mkdir -p ~/bin && \
    echo "export PATH=$PATH:~/bin" >> ~/.bashrc && \
    source ~/.bashrc && \
    curl -f -L -o "geth-pocr" \
    https://github.com/ethereum-pocr/go-ethereum/releases/download/master/geth && \
    chmod +x geth-pocr && mv geth-pocr ~/bin/geth
```

**3) download kerleano.json to `~/kerleano.json`**

Use same `kerleano_version` used by other nodes in the network
```sh
export kerleano_version=v1.0 && \
    curl -f -L -o ~/kerleano.json \
    https://github.com/ethereum-pocr/ethereum-pocr.github.io/releases/download/$kerleano_version/kerleano.json
```


**4) init node with kerleano.json**

```sh
# create the keystore folder out of the data folder
mkdir ~/.keystore
# directory where chaindata will be written, use external mount instead of ~/.ethereum/
export DATADIR=~/.ethereum/

geth init --datadir $DATADIR ~/kerleano.json

# move the nodekey outside of the data folder to keep the same node identity after a cleanup
mv $DATADIR/geth/nodekey ~/.keystore/nodekey
```

Make sure you see `Successfully wrote genesis state` in the log of the `geth init`command, otherwise you may have some data already written that should be deleted (rm datadir `rm -rf $DATADIR`)


**5) Get `kerleano` network `enodes`**

Get one or more `enodes` from here https://github.com/ethereum-pocr/kerleano/wiki/Kerleano-network-infos and export in env variable BOOTNODE.
will be used with `--bootnodes` option when starting the node

```sh
# write one or more enodes to a file
echo "<enode_url_1>" >> ~/.enodes
echo "<enode_url_xx>" >> ~/.enodes
# export one or more enodes in the variable BOOTNODE, delimiter is comma`
# export BOOTNODE=$(readarray -t ARRAY < ~/.enodes; IFS=','; echo "${ARRAY[*]}")
```

**6) Generate sealer account**

Generate an account and a passphrase to unlock it
```sh
# generate the account, will generate a public key and keystore in ~/.ethereum/keystore/
geth account new --keystore ~/.keystore

# export the Public address of the key (use)
export address=public_address_of_generated_account

#write the passphrase to a file
echo "replace_with_your_passphrase" > ~/.passphrase

```

**7) Start the sealer of the network**

```sh
# get public ip address
export PUBLIC_IP=$(curl -s ifconfig.me/ip)

# create the start script
echo 'BOOTNODE=$(readarray -t ARRAY < ~/.enodes; IFS=','; echo "${ARRAY[*]}")' > ~/start_sealer_node.sh
echo 'DATADIR=~/.ethereum/' >> ~/start_sealer_node.sh
echo 'KEYSTORE=~/.keystore/' >> ~/start_sealer_node.sh
echo 'NODEKEY=$KEYSTORE/nodekey' >> ~/start_sealer_node.sh
echo "address=$address" >> ~/start_sealer_node.sh
echo "PUBLIC_IP=$(curl -s ifconfig.me/ip)" >> ~/start_sealer_node.sh
echo 'nohup geth --networkid 1804 \
    --datadir $DATADIR \
    --bootnodes $BOOTNODE \
    --nodekey $NODEKEY \
    --syncmode full \
    --mine --miner.gasprice 1000000000 \
    --miner.etherbase $address --unlock $address \
    --password ~/.passphrase --keystore $KEYSTORE \
    --nat extip:$PUBLIC_IP \
    2>&1 1>>/tmp/eth.log &' >> ~/start_sealer_node.sh

chmod +x ~/start_sealer_node.sh


# run the node

~/start_sealer_node.sh
```


**8) Publish the `enode` of the sealer node**

```sh
# Get the enode of the sealer
geth attach --datadir $DATADIR --exec admin.nodeInfo.enode
```

then add the `enode` to this wiki page https://github.com/ethereum-pocr/kerleano/wiki/Kerleano-network-infos

**9) Authorize the node to start sealing**

At this stage the node is connected to the network but not able to seal blocks, create an issue with your public address and your identity and your motivation ot join the network here https://github.com/ethereum-pocr/kerleano/issues and wait for other nodes to authorize yours.

For the `clique`consensus you need more then 50% voters to join the network as sealer,

If your node is already sealer in the network, then you can vote to allow for new joiners,
once more then 50% of the nodes in the network do the commands bellow, the node will start sealing blocks
```sh
# ssh to the sealer node
ssh user@sealer_node
# attach a console to the sealer node
geth attach --datadir $DATADIR

# vote for a wallet to join the network
clique.propose("public_address_want_to_allow", true)
```

# Start client node

**1) configure VM2**

Repeat steps from 1) to 5) used for first sealer node
your client node should be init with the same genesis `kerleano.json`


**2) Start client node**

```sh
# get public ip address
export PUBLIC_IP=$(curl -s ifconfig.me/ip)

# create the start script
echo 'BOOTNODE=$(readarray -t ARRAY < ~/.enodes; IFS=','; echo "${ARRAY[*]}")' > ~/start_client_node.sh
echo 'DATADIR=~/.ethereum/' >> ~/start_client_node.sh
echo 'NODEKEY=~/.keystore/nodekey' >> ~/start_client_node.sh
echo "PUBLIC_IP=$(curl -s ifconfig.me/ip)" >> ~/start_client_node.sh
echo 'nohup geth --networkid 1804 \
    --datadir $DATADIR \
    --nodekey $NODEKEY \
    --bootnodes $BOOTNODE \
    --syncmode full \
    --http --http.addr=0.0.0.0 --http.port=8545 --http.api=web3,eth,net --http.corsdomain=* --http.vhosts=* \
    --ws --ws.addr=0.0.0.0 --ws.port=8546 --ws.api=web3,eth,net --ws.origins=* \
    --nat extip:$PUBLIC_IP \
    2>&1 1>>/tmp/eth.log &' >> ~/start_client_node.sh

chmod +x ~/start_client_node.sh


# run the node

~/start_client_node.sh
````

**3) Publish the `enode` of the client node**

```sh
# Get the enode of the client
geth attach --datadir $DATADIR --exec admin.nodeInfo.enode
```
then add the `enode` to `enodes` list in this wiki page https://github.com/ethereum-pocr/kerleano/wiki/Kerleano-network-infos

Client node expose http at port `8545` (by the option `--http.port 8545`) and websocket at `8546` (option `--ws.port 8546`)
So update in the same wiki page add in `rpc`list with `http://$PUBLIC_IP:8545/` and websockets list with `ws://$PUBLIC_IP:8546/`


# Assign a name to your node (optional)

you can simply edit the startup scripts `start_sealer_node.sh` and `start_client_node.sh` to add the option `--miner.extradata "The name of your node"` and give your node the name of your choice
