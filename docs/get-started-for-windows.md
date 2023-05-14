This is provided thanks to [André Rochet](https://github.com/andrerochet).

# Tutorial: Setting up Kerleano PoCR Node using Git Bash on Windows

This tutorial will guide you through the process of setting up the Kerleano Proof of Custody and Retrieval (PoCR) Node on a Windows PC using Git Bash. Git Bash is a shell interface for Git command-line features, it allows us to use various Linux command-line utilities in a Windows environment.

## **Prerequisites**
- Make sure you have Git Bash installed on your Windows PC.
- An internet connection to clone repositories and download necessary files.

**Important Note:** The exact paths used in this tutorial may differ from your personal setup. Replace `/path/to/your/workspace` with the correct path on your local machine.

## **Chapter 1: Cloning the Repository**

This step involves getting a copy of the Ethereum PoCR repository onto your local machine for us to work with. This is achieved using the `git clone` command.

### 1.1 Navigating to your workspace

First, we need to navigate to the folder where we want to keep our project. For this, we use the `cd` command, which stands for "change directory". Open Git Bash and navigate to your workspace using the `cd` command.

```bash
$ cd /path/to/your/workspace
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/c617a7f8-3b02-4520-ad28-3d421ce8197a)

### 1.2 Cloning the Repository

Now that we're in the right folder, we can get a copy of the repository. We do this using the `git clone` command. 

```bash
$ git clone --depth=1 https://github.com/ethereum-pocr/go-ethereum.git
```

**Expected Output:**

```bash
Cloning into 'go-ethereum'...
remote: Enumerating objects: 2102, done.
remote: Counting objects: 100% (2102/2102), done.
remote: Compressing objects: 100% (1802/1802), done.
remote: Total 2102 (delta 293), reused 1314 (delta 173), pack-reused 0
Receiving objects: 100% (2102/2102), 13.12 MiB | 10.58 MiB/s, done.
Resolving deltas: 100% (293/293), done.
Updating files: 100% (1854/1854), done.
```

The output above indicates that the repository has been successfully cloned into your workspace.
![image](https://github.com/andrerochet/kerleano/assets/133041646/cc2165d9-03cf-4db5-a2ef-1994aa10bb87)

## **Chapter 2: Building Geth Binary**

In this step, we will build a binary for Geth (Go Ethereum), which is a command-line interface for running a full Ethereum node.

### 2.1 Navigate into the cloned repository

To do this, we need to move into the cloned repository. We can do this with the `cd` command.

```bash
$ cd /path/to/your/workspace/go-ethereum
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/9aae67ba-35bf-4c4b-b379-23b643480084)

### 2.2 Building Geth

To build the Geth binary, we use the `make` command.

```bash
$ make geth
```

**Expected Output:**

```bash
env GO111MODULE=on go run build/ci.go install ./cmd/geth
>>> "C:\\Program Files\\Go\\bin\\go" build -ldflags "-X main.gitCommit=d1d5193135c353184bc8e3eed8c7d9130d0857f4 -X main.gitDate=20230303" -tags urfave_cli_no_docs -trimpath -v -o /path/to/your/workspace/go-ethereum/build/bin/geth.exe ./cmd/geth
Done building.
Run "./build/bin/geth" to launch geth.
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/2ee2f5be-6c27-49dd-bbf0-6d70dc76d149)

The output above indicates that the Geth binary has been successfully built.

## **Chapter 3: Setting up the Node**

Now, we are going to set up the node by creating a directory for the binary and making sure the system knows where to find it.

### 3.1 Creating a bin directory and copying geth

Return to your workspace with the `cd` command, create a directory named "bin" using the `mkdir` command (it stands for "make directory"), and then copy the `geth` binary into this new directory using the `cp` command (which stands for "copy").

```bash
$ cd /path/to/your/workspace
$ mkdir -p bin
$ cp go-ethereum/build/bin/geth bin/geth
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/4f26ba68-727a-4690-9ea7-e0955a36c6bd)

![image](https://github.com/andrerochet/kerleano/assets/133041646/df7a4483-5a20-4c9e-b969-289221c7fe01)

![image](https://github.com/andrerochet/kerleano/assets/133041646/5b71e769-f431-4487-890c-af3faac9d8e5)

### 3.2 Updating the PATH

Now, we need to let the system know where to find the `geth` binary. This is done by adding its directory to the PATH environment variable using the `export` command.

```bash
$ export PATH=$PATH:/path/to/your/workspace/bin
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/df6d9f7d-d761-43b8-863b-fbaea892811d)

To confirm that the `geth` command is now available, use the `which` command. This command shows the location of the binary associated with the provided command.

```bash
$ which geth
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/eee2f630-3702-4040-b07c-3640db53cd5a)

**Expected Output:**

```bash
/path/to/your/workspace/bin/geth
```

This output indicates that the `geth` binary is available in your PATH.

### 3.3 Downloading the Genesis File

The Genesis file is a JSON file which defines the initial state of your blockchain. We'll download it using the `curl` command.

```bash
$ curl -o genesis.json -L https://github.com/ethereum-pocr/ethereum-pocr.github.io/releases/download/v2.0.0/kerleano.json
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/68b65b32-3727-4e4d-8b27-672b5ae16f32)

### 3.4 Downloading the Bootnodes File

The Bootnodes file is a list of nodes in the network that our node can initially connect to. We'll also download this using the `curl` command.

```bash
$ curl -o bootnodes -L https://raw.githubusercontent.com/ethereum-pocr/kerleano/main/BOOTNODES
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/539678e0-70f8-4bcf-b7fc-205480cdf938)

## **Chapter 4: Initializing the Node**

With all the necessary files and binaries in place, it's time to initialize our node.

### 4.1 Initialize Geth with the Genesis File

We'll initialize our Geth node with the Genesis file we downloaded. The `--datadir` option specifies the directory where the blockchain data will be stored.

```bash
$ ./bin/geth init --datadir .ethereum ./genesis.json
```

**Expected Output:**

```bash
INFO [05-10|12:34:19.785] Persisted trie from memory database      nodes=13 size=1.80KiB time=0s gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [05-10|12:34:19.786] Successfully wrote genesis state         database=chaindata hash=23dafd..0f9e5d
INFO [05-10|12:34:19.786] Allocated cache and file handles         database=/path/to/your/workspace/.ethereum/geth/lightchaindata cache=16.00MiB handles=16
INFO [05-10|12:34:19.840] Opened ancient database                  database=/path/to/your/workspace/.ethereum/geth/lightchaindata/ancient/chain readonly=false
INFO [05-10|12:34:19.840] Writing custom genesis block
INFO [05-10|12:34:19.840] Persisted trie from memory database      nodes=13 size=1.80KiB time=0s gcnodes=0 gcsize=0.00B gctime=0s livenodes=1 livesize=0.00B
INFO [05-10|12:34:19.842] Successfully wrote genesis state         database=lightchaindata hash=23dafd..0f9e5d
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/2f5ce113-945b-4afa-b31e-747e7c45b4a5)

This output indicates that the Geth node has been successfully initialized with the provided Genesis file, and the data directories have been configured.

## **Chapter 5: Running the Node**

Finally, it's time to start running our node!

### 5.1 Setting Up the Environment Variables

In this step, we will be setting up some necessary environment variables. These are used by the `geth` command to configure your node.

```bash
$ BOOTNODES=$(readarray -t ARRAY < bootnodes; IFS=,; echo "${ARRAY[*]}") # convert the file into a comma separated list
$ NETWORK_ID=$(grep chainId genesis.json | grep -Eo '[0-9]+') # get the value of the chain Id and use it as network id as well
$ PUBLIC_IP=$(curl -s ifconfig.me/ip) # needed to expose your node to other, else you will not be reachable in discovery
$ SYNCMODE=snap # tells to synchronize without validating the transactions (faster but imply you trust the bootnodes)
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/a694274a-894d-48ef-9887-245d9c469f35)

### 5.2 Launching the Nodes

Now you can start your node with the `geth` command. The `--networkid` option is used to specify the network id. `--datadir` option is used to specify the data directory. The `--bootnodes` option is used to specify the bootnodes. The `--syncmode` option is used to specify the sync mode.

```bash
$ ./bin/geth --networkid $NETWORK_ID --datadir .ethereum --bootnodes "$BOOTNODES" --syncmode $SYNCMODE --http --http.addr=0.0.0.0 --http.port=8545 --http.api=web3,eth,net,clique --http.corsdomain=* --http.vhosts=* --ws --ws.addr=0.0.0.0 --ws.port=8546 --ws.api=web3,eth,net,clique --ws.origins=* --nat extip:$PUBLIC_IP
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/1c57026e-a15c-4b58-b4dc-582ccc7ac673)

Wait until the network sync (can take up to 15+ minutes depending on when you sync).

**Now Make sure you keep Gitbash opened and running and you open a second gitbash for the rest of the tutorial.**

## **Chapter 6: Script**

**6.1 Create a Startup Script**

Let's pretend this is a recipe we're making. We'll put the instructions into a file named `start_node.sh`.

**Step 1: Open Git Bash**

First, open your Git Bash application. This is a program where you can type commands to your computer. 

**Step 2: Create a new file**

We're going to create a new text file to hold our recipe. In Git Bash, type the following command and then press enter:

```
touch start_node.sh
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/07d597c4-2986-42be-bd98-3cc5e5e0d082)

This is like saying to your computer, "Hey, make me a new blank file named `start_node.sh`."

![image](https://github.com/andrerochet/kerleano/assets/133041646/dab94e26-c1d7-4a11-80d0-ae053ca27f8e)

What it looks like on your computer

**Step 3: Open the file in a text editor**

Next, we'll open our new file in a text editor so we can write in it. Type the following command and press enter:

```
nano start_node.sh
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/5d3c9856-ef21-470b-b242-6221e607e860)

This tells your computer, "I want to write in the `start_node.sh` file, please open it for me."

**Step 4: Write the recipe**

Now, we're going to write down our recipe. In the nano editor, type or copy-paste the following:

```bash
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
![image](https://github.com/andrerochet/kerleano/assets/133041646/eb308c30-c5ad-44e8-8832-011b267b06bc)

This is like the cooking instructions in a recipe.

**Step 5: Save and close the file**

Once you've typed all that in, you need to save your recipe. In nano, you do this by pressing `Ctrl + O`, then `Enter` to confirm. To close nano, press `Ctrl + X`.

![image](https://github.com/andrerochet/kerleano/assets/133041646/9a10af3f-6a00-4544-b95a-9cac33d3d434)
After pressing `Ctrl + O`

![image](https://github.com/andrerochet/kerleano/assets/133041646/a05bb341-83e4-4b2e-8bee-e97515ecdb9a)
After pressing `Enter`

![image](https://github.com/andrerochet/kerleano/assets/133041646/869c58b4-cdeb-40f3-862f-a55b584ae175)
After pressing `Ctrl + X`

**Step 6: Make the file executable**

Finally, we need to tell our computer that it's safe to use this recipe. Type the following command and press enter:

```
chmod +x start_node.sh
```
![image](https://github.com/andrerochet/kerleano/assets/133041646/f35bade8-2e7f-47c6-9b7e-ab2204d7aaec)

This is like putting a seal of approval on your recipe, saying "This is safe to cook."

**6.2 Connecting and Interacting with the Node**

Now that your node is up and running, you can start talking to it! The node has two places where you can chat: an HTTP (port 8545) and a Websocket (port 8546). These are like the node's ears, and they're listening for your instructions.

To start a conversation with your node, try saying "Hello!" using these commands:

```bash
geth attach http://localhost:8545
# or
geth attach ws://localhost:8546
```
Now you're talking to your node! You can also connect your Metamask (or any compatible wallet) to your node, just like adding a friend on a social network.

![image](https://github.com/andrerochet/kerleano/assets/133041646/00236b90-f307-432d-ae27-85e8baf52031)

## **Chapter 7: Join the consensus**

**7.0 Turning Your Node into a Sealer**

Now let's take a big step forward: turning your node into a sealer.

**7.1 Creating the Sealer Etherbase Wallet**

A sealer is like a trusted messenger in the network. It needs to have a special key, a private key, to sign the blocks. This key is identified by a public address, also known as miner.etherbase or coinbase. 

Creating this key is like making a special signature for your sealer. Geth provides a tool to create this signature in the Web3 Secret Storage format. Run this command:

```bash
geth account new --keystore .keystore 
```

![image](https://github.com/andrerochet/kerleano/assets/133041646/ddd3a967-d77c-4d46-934d-c1b087003c93)

When you run this command, it will ask you for a password. Remember, this password is super secret, don't forget it, and never share it with anyone!

Once you've got your password, save it .

![image](https://github.com/andrerochet/kerleano/assets/133041646/d570f03e-08c1-462c-9574-42e97a4a3b20)

**7.2 Modifying the Script to Act as a Sealer**

Now, we need to modify our recipe (the startup script we talked about earlier) to make our node act as a sealer. It's like adding some secret ingredients to our cake recipe.

Okay, let's update the 'start_node.sh' file without exiting Git Bash:

1. **Open Git Bash**: If it's not already open, find the Git Bash application on your computer and open it.

2. **Navigate to the correct directory**

3. **Create or Update the 'start_node.sh' file**: You can use the `echo` command in Git Bash to write the new content into 'start_node.sh'. Here's how:

Type the following commands one line at a time, hitting enter after each line:

```bash
echo '#!/bin/bash' > start_node.sh

echo 'BOOTNODES=$(readarray -t ARRAY < bootnodes; IFS=,; echo "${ARRAY[*]}")' >> start_node.sh

echo 'NETWORK_ID=$(grep chainId genesis.json | grep -Eo '[0-9]+')' >> start_node.sh

echo 'PUBLIC_IP=$(curl -s ifconfig.me/ip)' >> start_node.sh

echo 'SYNCMODE=full' >> start_node.sh

echo 'ETHERBASE=0x64d0eFdeFb40E39A5ea946A0E132Dc0b690BD2c1' >> start_node.sh

echo 'nohup geth --networkid $NETWORK_ID --datadir .ethereum --bootnodes "$BOOTNODES" --syncmode $SYNCMODE \' >> start_node.sh

echo '--keystore .keystore --miner.etherbase $ETHERBASE --unlock $ETHERBASE --password .keystore/password \' >> start_node.sh

echo '--miner.extradata="Your name as sealer (32b max)" \' >> start_node.sh

echo '--mine --miner.gasprice 1000000000 \' >> start_node.sh

echo '--nat extip:$PUBLIC_IP 2>&1 1>> geth.log &' >> start_node.sh
```
The `>` character tells Git Bash to write the content that follows into 'start_node.sh', replacing any existing content. The `>>` characters tell Git Bash to append the content that follows to 'start_node.sh', adding to the existing content instead of replacing it.

Remember, replace `ETHERBASE` with the address of the wallet you created earlier.

![image](https://github.com/andrerochet/kerleano/assets/133041646/79f44d94-c1f5-40d4-8551-1f34f655fb88)

4. **Check the contents**: To make sure the file was updated correctly, you can display its contents using the `cat` command: `cat start_node.sh`.

![image](https://github.com/andrerochet/kerleano/assets/133041646/2f14902c-3f4d-4617-be29-3a567d2315d1)


**7.3 Requesting to Join the Consensus**

After you've done all this, you might notice that you're not able to sign blocks yet. This is because you have to be accepted as a sealer in the network first, just like being accepted into a club.

To join the club, you have to make a [request](https://github.com/ethereum-pocr/kerleano/issues). You do this by submitting an issue with your sealer address (the one you created earlier). You also have to tell everyone who you are and why you want to join, just like introducing yourself at a party.

Once you're accepted by a majority of the sealers, your node will start creating blocks. It's like being given the power to contribute to the network. This process is a bit more strict on the real network, which is explained in the whitepaper.

To connect to your node after you've been accepted, use this command:

```bash
./geth attach .ethereum/geth.ipc 
```

And that's it! You've turned your node into a sealer and are now part of the consensus. Congratulations!
