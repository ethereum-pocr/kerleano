# Kerleano

***Network infos***

  *  Use **[Issues](https://gitlab.com/saturnproject/externalgrp/pocr/kerleano/-/issues)** to submit a request to joint kerleano network or read the existing requests

  *  Use **[Network infos](https://gitlab.com/saturnproject/externalgrp/global_qna/-/wikis/Networks-infos)** to get `enodes` url, `rpc`and `websocket` endpoints to join and interact with the network

***Nodes topology***


- **Sealer node**: is an etheruem node that will be authorized to seal blocks and get CTC as rewards once the node  is monitored by an auditor.
The sealer node will generate an account (public and private keys) and his public key (wallet or address) should be authorized by the other sealers of the network (using `clique.propose("address", true)` POA clique consensus)
The network `kerleano` is initilized with a given address `0x6e45c195e12d7fe5e02059f15d59c2c976a9b730` (see [How kerleano network is initilized](docs/init_Kerleano_network.md)). 
Once the network is set, this address can be removed (using `clique.propose("address", false)` POA clique consensus)

- **Client node**: is a readonly etheruem node that will read all the blocs of the network and expose RPC and Websocket endpoints in order to let apps (monitoring, bloc explorers, front wallet...) get infos about the network


***Join the network***

  * See [Join kerleano network](docs/join_Kerleano_network.md)
