# To join the Kerleano network quickly

and get started to submit transactions and read the chain you can use the preconfigured docker image

```sh
docker run --rm -it -v $(pwd)/data:/app/.ethereum -p 8545:8545 -p 8546:8546 ghcr.io/ethereum-pocr/go-ethereum/client-kerleano:2.0.0 
```
