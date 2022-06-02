# Ansible scripts to facilitate joining the kerleano network

Joining the kerleano network can be quite simply scripted in ansible. The following is a guide to setting up sealer and client nodes on remote machines.

## A note on user permissions

For security reasons, all geth processes should be executed by a user having no sudo rights to prevent
eventually malicious code from gaining access to the machine and manipulating account information. For this reason,
the scripts presented here use a specific user, `kerleano_user` for all `geth` processes.

## Variables

Variables used in the following scripts are as follows:

- `installation_dir` : the directory used for installation
- `geth_url` : the url of the geth package hosted on the saturn repo
- `kerleano_url`: the url of the kerleano configuration (json file)
- `kerleano_user`: the user for installation and subsequent execution
- `enodes`: a list of enode addresses that will be used for connection to the network (provided in the saturn kerleano repo wiki)

## Common configuration for client and sealer nodes

Both sealer and client nodes share a base installation that involves downloading the `geth` executable and `kerleano` configuration. The user should have access to the packages through `GITLAB_USER` and `GITLAB_ACCESS_TOKEN` environment variables. A gitlab access token can be generated through gitlab in the user preferences page.

Base installation can be effected as follows. The script will generate a nodekey and store it in the `.keystore` folder.

```yaml
- name: make subdirectories on installation path
  file:
    path: "{{installation_dir}}/{{ item }}"
    state: directory
    owner: "{{ kerleano_user }}"
  loop:
    - bin
    - .keystore
    - .ethereum
- name: download geth executable
  get_url:
    url: "{{ geth_url }}"
    dest: "{{ installation_dir }}/bin/geth"
    username: "{{ lookup('env', 'GITLAB_USER') }}"
    password: "{{ lookup('env', 'GITLAB_ACCESS_TOKEN') }}"
    mode: +x
- name: download kerleano json configuration
  get_url:
    url: "{{ kerleano_url }}"
    dest: "{{ installation_dir }}/kerleano.json"
    username: "{{ lookup('env', 'GITLAB_USER') }}"
    password: "{{ lookup('env', 'GITLAB_ACCESS_TOKEN') }}"
- name: initialize geth
  shell: bin/geth init --datadir .ethereum kerleano.json
  args:
    chdir: "{{ installation_dir }}"
- name: move nodekey to keystore
  shell: "mv .ethereum/geth/nodekey .keystore/nodekey"
  args:
    chdir: "{{ installation_dir }}"
```


## Initialize sealer account

Once the base installation has been done, you're ready to generate a sealer account.

Start by generating a password, which will be stored in a `.passphrase` file.
Initialize the geth account with this password and extract the generated address into an `address` file.

This can be done as follows. Once the script has been executed, you should have a startup script in your installation directory.

```yaml
- name: generate geth password
  shell: echo $(date +%s.%N | sha256sum | base64 | head -c 32) > .passphrase
  args:
    chdir: "{{ installation_dir }}"
- name: initialize new sealer account
  shell: bin/geth --password .passphrase account new --keystore .keystore > account_info
  args:
    chdir: "{{ installation_dir }}"
- name: generate address file
  shell: echo $(cat account_info | grep "key:" | awk '{print $6}') > address
  args:
    chdir: "{{ installation_dir }}"
- name: generate the startup script
  copy:
    dest: "{{ installation_dir }}/start_sealer_node.sh"
    mode: +x
    content: |
      #!/bin/bash
      cd {{installation_dir}}
      bootnodes={{ enodes | join(',') }}
      address=$(cat address)
      public_ip=$(curl -s ifconfig.me/ip)
      bin/geth --networkid 1804 --datadir .ethereum --bootnodes $bootnodes --nodekey .keystore/nodekey --syncmode full --mine --miner.gasprice 1000000000 --miner.etherbase $address --unlock $address --password .passphrase --keystore .keystore --nat extip:$public_ip
```

### Ask for the sealer address to be added to the network or reuse an existing wallet information

At this point, your sealer has an address and is ready to seal. This address will require an authorization within the network, which can be done through gitlab by creating an issue in the kerleano repo and providing reasons/motivation for wanting to join.

If your sealer has already been added to the network and you wish to reuse an existing, authorized wallet, you will want to override the generated nodekey, passphrase, address and wallet info.

The following script can be executed to reuse an existing wallet:

```yaml
- name: override the passphrase
  copy:
    dest: "{{installation_dir}}/.passphrase"
    content: "<your previous passphrase>"
- name: override the address
  copy:
    dest: "{{installation_dir}}/address"
    content: "<your previous address>
- name: override the nodekey
  copy:
    dest: "{{installation_dir}}/.keystore/nodekey"
    content: "<your previous nodekey>"
- name: get the name of the wallet file
  find:
    paths: "{{ installation_dir }}/.keystore"
    patterns: UTC*
  register: wallet_file_name
- name: override the wallet configuration
  copy:
    dest: "{{wallet_file_name.files[0].path}}"
    content: "<the previous content to be found in .keystore/UTC*>"
```

## Set up client nodes

Client nodes use a different configuration than sealer nodes and do not require setting up a geth account.
As such, generating the startup script is simpler.

Generating a client node startup script can be done as follows:

```yaml
- name: generate the startup script
  copy:
    dest: "{{ installation_dir }}/start_client_node.sh"
    mode: +x
    content: |
      #!/bin/bash
      cd {{installation_dir}}
      bootnodes={{ enodes | join(',') }}
      address=$(cat address)
      public_ip=$(curl -s ifconfig.me/ip)
      bin/geth --networkid 1804 --datadir .ethereum --nodekey .keystore/nodekey --bootnodes $bootnodes --syncmode full --http --http.addr=0.0.0.0 --http.port=8545 --http.api=web3,eth,net --http.corsdomain=* --http.vhosts=* --ws --ws.addr=0.0.0.0 --ws.port=8546 --ws.api=web3,eth,net --ws.origins=* --nat extip:$public_ip
```

## Executing sealer and client nodes

Execution of the sealer and client nodes should be setup as services on the remote machines. This will facilitate log management and allow for the processes to resume on a unexpected or planned reboot.
