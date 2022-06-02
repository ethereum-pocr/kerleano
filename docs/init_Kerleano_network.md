# Bootstrap the network `kerleano`

In this example we are using 2 `ubuntu` VMs to bootstrap the network `kerleano`

**VM1: will be the first sealer in the network, and will use a pre-generated and authorized address**
- VM1 requirements:
    * os: Ubuntu 20.04.4 LTS
    * ram: 2Gb
    * cpu: ??
    * tcp/udp port 30303: default geth node (can override by `--port` when starting the node)

**VM2 : will be a client node and expose the rpc and ws endpoints of the network**
- VM1 requirements:
    * os: Ubuntu 20.04.4 LTS
    * ram: 2Gb
    * cpu: ??
    * tcp/udp port 30303: default geth port (can override by `--port` when starting the node)
    * port 8545: default rpc port (can override by `--http.port` when starting the node)
    * port 8546: default websocket port (can override by `--ws.port` when starting the node)

# Start first sealer node

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

```sh
export kerleano_version=v1.0 && \
    curl -f -L -o ~/kerleano.json \
    https://github.com/ethereum-pocr/ethereum-pocr.github.io/releases/download/$kerleano_version/kerleano.json
```


**4) init node with kerleano.json**

```sh
# directory where chaindata will be written, use external mount instead of ~/.ethereum/
export DATADIR=~/.ethereum/

geth init --datadir $DATADIR ~/kerleano.json
```

Make sure you see `Successfully wrote genesis state` in the log of the `geth init`command, otherwise you may have some data already written that should be deleted (rm datadir `rm -rf $DATADIR`)


**5) prepare the default public/private keys used by `kerleano`**

`kerleano` network use the address `0x6e45c195e12d7fe5e02059f15d59c2c976a9b730` as default authorized wallet in the network

and the private key of this address is the json bellow
***Attention! never use this address in production, use only to bootstrap the network and then use `clique`consensus to remove it from the network when the other nodes have properly been setup***
```javascript
{"address":"6e45c195e12d7fe5e02059f15d59c2c976a9b730","crypto":{"cipher":"aes-128-ctr","ciphertext":"d3843d290994fbbdfaeddb9b049d35879af7ac68113210b466f936427f2ff263","cipherparams":{"iv":"eebb6604899c19c5cbab122e8d7aafc4"},"kdf":"scrypt","kdfparams":{"dklen":32,"n":262144,"p":1,"r":8,"salt":"18bec5873f021f6dadbe93ca90154832d831bee5f528342b4cfa3c3190f99036"},"mac":"f908a7daa8e98323a360112c589b1e5496e8cf59125835a19eb485452eff2431"},"id":"d6f4754c-1aa6-42b4-94d0-71eb4a072743","version":3}
```

```sh
# export address (will be used when starting sealer node)
export address=0x6e45c195e12d7fe5e02059f15d59c2c976a9b730

# write passphrase file used for the private key with empty content
touch ~/.passphrase
# write the private address in a file
mkdir ~/keystore/

echo '{"address":"6e45c195e12d7fe5e02059f15d59c2c976a9b730","crypto":{"cipher":"aes-128-ctr","ciphertext":"d3843d290994fbbdfaeddb9b049d35879af7ac68113210b466f936427f2ff263","cipherparams":{"iv":"eebb6604899c19c5cbab122e8d7aafc4"},"kdf":"scrypt","kdfparams":{"dklen":32,"n":262144,"p":1,"r":8,"salt":"18bec5873f021f6dadbe93ca90154832d831bee5f528342b4cfa3c3190f99036"},"mac":"f908a7daa8e98323a360112c589b1e5496e8cf59125835a19eb485452eff2431"},"id":"d6f4754c-1aa6-42b4-94d0-71eb4a072743","version":3}' > ~/keystore/pkey_6e45c195e12d7fe5e02059f15d59c2c976a9b730


```

**6) Start the first sealer of the network**

```sh
# get public ip address
export PUBLIC_IP=$(curl -s ifconfig.me/ip)

# start geth node
echo 'nohup geth --networkid 1804 \
    --datadir $DATADIR \
    --syncmode full \
    --mine --miner.gasprice 1000000000 \
    --miner.etherbase $address --unlock $address \
    --password ~/.passphrase --keystore ~/keystore/ \
    --nat extip:$PUBLIC_IP \
    2>&1 1>>/tmp/eth.log &' > ~/start_sealer_node.sh && \
    chmod +x ~/start_sealer_node.sh && \
    ~/start_sealer_node.sh
```

**7) Publish the `enode` of the sealer node**

```sh
# Get the enode of the sealer
geth attach --datadir $DATADIR --exec admin.nodeInfo.enode
```

then add the `enode` to this wiki page https://github.com/ethereum-pocr/kerleano/wiki/Kerleano-network-infos

# Start client node

**1) configure VM2**

Repeat steps from 1) to 4) used for first sealer node
your client node should be init with the same genesis `kerleano.json`

**2) Get `kerleano` network `enodes`**

Get one or more `enodes` from here https://github.com/ethereum-pocr/kerleano/wiki/Kerleano-network-infos and export in env variable BOOTNODE.
will be used with `--bootnodes` option when starting the node

```sh
# write one or more enodes to a file
echo "<enode_url_1>" >> ~/.enodes
echo "<enode_url_xx>" >> ~/.enodes
# export one or more enodes in the variable BOOTNODE, delimiter is comma`
export BOOTNODE=$(readarray -t ARRAY < ~/.enodes; IFS=','; echo "${ARRAY[*]}")
```

**3) Start client node**

```sh
# get public ip address
export PUBLIC_IP=$(curl -s ifconfig.me/ip)
# export default address authorized by kerleano
export address=0x6e45c195e12d7fe5e02059f15d59c2c976a9b730

# start geth node
echo 'nohup geth --networkid 1804 \
    --bootnodes $BOOTNODE \
    --datadir $DATADIR \
    --http --http.addr "0.0.0.0" --http.port 8545 \
    --http.api "eth,web3,net,admin,debug,personal" --http.corsdomain "*" \
    --ws --ws.addr "0.0.0.0" --ws.port 8546 \
    --ws.api "eth,web3,net,admin,debug,personal" --ws.origins "*" \
    --syncmode full \
    --nat extip:$PUBLIC_IP \
    --miner.etherbase $address \
    2>&1 1>>/tmp/eth.log &' > ~/start_client_node.sh && \
    chmod +x ~/start_client_node.sh && \
    ~/start_client_node.sh
```

**4) Publish the `enode` of the client node**

```sh
# Get the enode of the client
geth attach --exec admin.nodeInfo.enode
```
then add the `enode` to `enodes` list in this wiki page https://github.com/ethereum-pocr/kerleano/wiki/Kerleano-network-infos

Client node expose http at port `8545` (by the option `--http.port 8545`) and websocket at `8546` (option `--ws.port 8546`)
So update in the same wiki page add in `rpc`list with `http://$PUBLIC_IP:8545/` and websockets list with `ws://$PUBLIC_IP:8546/`
