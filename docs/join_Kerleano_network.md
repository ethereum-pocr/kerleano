# Join the network `kerleano`

In this example we are using 2 `ubuntu` VMs to bootstrap the network `kerleano`

**VM1: will be the sealer in the network**
- VM1 requirements: 
    * os: ubuntu
    * ram: 2Gb
    * cpu: ??
    * tcp/udp port 30303: default geth node (can override by `--port` when starting the node)

**VM2 : will be a client node and expose the rpc and ws endpoints of the network**
- VM1 requirements: 
    * os: ubuntu
    * ram: 2Gb
    * cpu: ??
    * tcp/udp port 30303: default geth port (can override by `--port` when starting the node)
    * port 8545: default rpc port (can override by `--http.port` when starting the node)
    * port 8546: default websocket port (can override by `--ws.port` when starting the node)

# Start sealer node

**1) ssh into the VM**

```sh
ssh user@VM1
```
 
**2) export gitlab user and access token with read_api scope**<br>

Use gitlab personal page to create the token: https://gitlab.com/-/profile/personal_access_tokens.
Your GITLAB_USER is the user without the @ or alternatively the name of the token you created.

```sh
export GITLAB_USER=your_gitlab_user
export GITLAB_ACCESS_TOKEN=your_access_token_with_read_api
```

**3) download geth binary, give exec right and move to /usr/bin/**

```sh
# download the latest released stable version (in this example we go for `latest`)
export geth_version=latest && \
    sudo curl -f -u "${GITLAB_USER}:${GITLAB_ACCESS_TOKEN}" -o "geth-pocr" \
    https://gitlab.com/api/v4/projects/34464473/packages/generic/geth/${geth_version}/geth && \
    sudo chmod +x geth-pocr && sudo mv geth-pocr /usr/bin/geth
```

**4) download kerleano.json to `~/kerleano.json`**

```sh
export kerleano_version=latest && \
    sudo curl -f -u "${GITLAB_USER}:${GITLAB_ACCESS_TOKEN}" -o ~/kerleano.json \
    https://gitlab.com/api/v4/projects/34381428/packages/generic/genesis/${kerleano_version}/kerleano.json
```


**5) init node with kerleano.json**

```sh
geth init --datadir ~/.ethereum/ ~/kerleano.json
```

Make sure you see `Successfully wrote genesis state` in the log of the `geth init`command, otherwise you may have some data already written that should be deleted (default datadir `rm -rf ~/.ethereum/`)


**6) Generate sealer account**

Generate an account and a passphrase to unlock it
```sh
# generate the account, will generate a public key and keystore in ~/.ethereum/keystore/
geth account new

# export the Public address of the key (use)
export address=public_address_of_generated_account

#write the passphrase to a file
echo "replace_with_your_passphrase" > ~/.passphrase

```

**7) Get `kerleano` network `enodes`**

Get one or more `enodes` from here https://gitlab.com/saturnproject/externalgrp/global_qna/-/wikis/Networks-infos and export in env variable BOOTNODE.
will be used with `--bootnodes` option when starting the node

```sh
# export one or more enodes in the variable BOOTNODE, delimiter is space`
export BOOTNODE="<get_enodes_space_separated>"
```

**8) Start the sealer of the network**

```sh
# get public ip address
export PUBLIC_IP=$(curl -s ifconfig.me/ip)

# start geth node 
nohup geth --networkid 1804 \
    --datadir ~/.ethereum/ \
    --bootnodes $BOOTNODE \
    --syncmode full \
    --mine --miner.gasprice 1000000000 \
    --miner.etherbase $address --unlock $address \
    --password ~/.passphrase --allow-insecure-unlock --keystore ~/.ethereum/keystore/ \
    --nat extip:$PUBLIC_IP \
    2>&1 1>>/tmp/eth.log &
```

**9) Publish the `enode` of the sealer node**

```sh
# Get the enode of the sealer
geth attach --exec admin.nodeInfo.enode
```

then add the `enode` to this wiki page https://gitlab.com/saturnproject/externalgrp/global_qna/-/wikis/Networks-infos

**10) Authorize the node to start sealing**

At this stage the node is connected to the network but not able to seal blocks, submit your public address with your identity here https://gitlab.com/saturnproject/externalgrp/global_qna/-/wikis/Identities and wait for other nodes to authorize the node.

For the `clique`consensus you need more then 50% voters to join the network as sealer,

If your node is already sealer in the network, then you can vote to allow for new joiners
once more then 50% of the nodes in the network do the commands bellow, the node will start sealing blocks
```sh
# ssh to the sealer node
ssh user@sealer_node
# attach a console to the sealer node
geth attach --datadir ~/.ethereum/

# vote for a wallet to join the network
clique.propose("public_address_want_to_allow", true)
```

# Start client node

**1) configure VM2**

Repeat steps from 1) to 5) used for first sealer node 
your client node should be init with the same genesis `kerleano.json`

**2) Get `kerleano` network `enodes`**

Get one or more `enodes` from here https://gitlab.com/saturnproject/externalgrp/global_qna/-/wikis/Networks-infos and export in env variable BOOTNODE.
will be used with `--bootnodes` option when starting the node

```sh
# export one or more enodes in the variable BOOTNODE, delimiter is space`
export BOOTNODE="<get_enodes_space_separated>"
```

**3) Start client node**

```sh
# get public ip address
export PUBLIC_IP=$(curl -s ifconfig.me/ip)

# start geth node 
nohup geth --networkid 1804 \
    --bootnodes $BOOTNODE \
    --datadir ~/.ethereum/ \
    --http --http.addr "0.0.0.0" --http.port 8545 \
    --http.api "eth,web3,net,admin,debug,personal" --http.corsdomain "*" \
    --ws --ws.addr "0.0.0.0" --ws.port 8546 \
    --ws.api "eth,web3,net,admin,debug,personal" --ws.origins "*" \
    --syncmode full \
    --nat extip:$PUBLIC_IP \
    --miner.etherbase $address \
    2>&1 1>>/tmp/eth.log &
```

**4) Publish the `enode` of the client node**

```sh
# Get the enode of the client
geth attach --exec admin.nodeInfo.enode
```
then add the `enode` to `enodes` list in this wiki page https://gitlab.com/saturnproject/externalgrp/global_qna/-/wikis/Networks-infos

Client node expose http at port `8545` (by the option `--http.port 8545`) and websocket at `8546` (option `--ws.port 8546`)
So update in the same wiki page add in `rpc`list with `http://$PUBLIC_IP:8545/` and websockets list with `ws://$PUBLIC_IP:8546/`







