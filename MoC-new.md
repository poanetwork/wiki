# ~~not-so~~ Easy steps to create a new network
(for the Master of Ceremony)

Prerequisites:
1. Plenty of time
2. Patience
3. New network's `NetworkID`
4. New network's `NetworkName`

## Chapter I - in which MoC creates his secret keys and never shows them to others

You will need to generate ethereum address and keystore file for the `owner`.  
One way is to download `etherwallet-v*.*.*.*.zip` archive of the latest release from myetherwallet github repo https://github.com/kvhnuke/etherwallet/releases/
then unplug your computer from the Internet, extract zip archive and open `index.html` in your browser.
Please be sure to use strong password, download keystore file `UTC--*--*`, your private key and keep them in a safe place.

## Chapter II - in which MoC forks first portion of repos and replaces some text

There are quite a few repositories that are used to run the network. You will need to fork them and later update parameters.
Please be consistent with naming of branches and use `NetworkName`.

### Configs on azure
https://github.com/oraclesorg/deployment-azure/tree/dev-mainnet
1. Create a separate branch named `NetworkName`
2. Open `nodes/bootnodes.txt` and remove all lines from this file
3. Don't change anything else, this repository is used only for configs now

### POA Network Consensus contract
https://github.com/oraclesorg/poa-network-consensus-contracts
1. Create a separate branch named `NetworkName`
2. Clone it to your local machine
3. Install `python3`, `pip3`, `solc`: **make sure to use binary package for solc, not the one from npm** http://solidity.readthedocs.io/en/develop/installing-solidity.html#binary-packages
4. Install `pip3 install solidity-flattener`
5. Run `npm install`
6. Run `./make_flat.sh` to generate flat versions of contracts. They will be saved to `flat/`
7. Open [Remix](http://remix.ethereum.org/) in your browser, copy-paste code from `flat/PoaNetworkConsensus_flat.sol`, press "Start to compile".
8. On "Run" tab select "Javascript VM" as environment, "PoaNetworkConsensus" as your contract, in "Create" field paste MoC's address "0x..." and click "Create"
9. After the contract is compiled click "Details" button and copy it's bytecode

### Chain.json
https://github.com/oraclesorg/oracles-chain-spec
1. Create a separate branch named `NetworkName`
2. In "params" block, change networkID to your `NetworkID` in hex.
3. Scroll down to "accounts" block and replace constructor for "0xf472e0e43570b9afaab67089615080cf7c20018d" with bytecode you obtained from POA Network Consensus contract "0x606060..."
4. Replace address of account with huge amount of money with your MoC address

### Ansible playbook
https://github.com/oraclesorg/deployment-playbooks
1. Create a spearate branch named `NetworkName`
2. Open `group_vars/all.network` and replace the following variables with corresponding branch names (should be `NetworkName` mostly)
* `SCRIPTS_MOC_BRANCH`
* `SCRIPTS_VALIDATOR_BRANCH`
* `TEMPLATES_BRANCH`
* `GENESIS_BRANCH`
3. Replace `MOC_ADDRESS` with your MoC address
4. If you forked repos to your own github account, also replace the `MAIN_REPO_FETCH` value with your account name.
5. You may also want to replace
* `NODE_SOURCE_DEB` - node.js version
* `PARITY_BIN_LOC` - url to parity binary

## Chapter III - in which MoC takes a deep breath and creates first nodes of the network
1. Install ansible
2. Clone https://github.com/oraclesorg/deployment-playbooks and `git checkout` to the correct branch.
3. Prepare files with your ssh public keys, e.g.
```
cat ~/.ssh/id_rsa.pub > files/admins.pub
cp files/admins.pub files/ssh_netstat.pub
cp files/admins.pub files/ssh_bootnode.pub
cp files/admins.pub files/ssh_explorer.pub
cp files/admins.pub files/ssh_moc.pub
cp files/admins.pub files/ssh_validator.pub
```

### Start with netstat server
1. Create a file with a full config for this node type:
```
cat group_vars/all.network group_vars/netstat.example > group_vars/all
```
2. Open this file and fill missing values at the end
3. Create an instance
```
ansible-playbook netstat.yml
```
Wait till the command completes, extract from logs and write down IP address and AWS InstanceID of the new node.

4. Create file `hosts` with the following content (assuming new node's IP is 1.2.3.4)
```
[netstat]
1.2.3.4
```

5. Configure the instance
```
ansible-playbook -i hosts site.yml -t netstat
```
If this command fails because host is unreachable over ssh, wait a minute and start again, it takes some time to reboot.

6. If you plan on using Cloudflare and/or SSL Certificates for netstat, it's time to configure them. Then
```
ssh root@1.2.3.4
```
stop nginx
```
service nginx stop
```
backup current certificates:
```
cp -R /etc/nginx/ssl /etc/nginx/ssl.orig
```
then replace the content of `/etc/nginx/ssl/server.crt` and `/etc/nginx/ssl/server.key/` with your certificate and private key. Then start nginx again
```
service nginx start
```
and check if it's running
```
ps aux | grep nginx
```

### Spawn bootnodes
1. Create a file with a full config for this node:
```
cat group_vars/all.network group_vars/bootnode.example > group_vars/all
```

2. Fill missing values in the end of the file. Use `https://netstat.server.com` for `NETSTAT_SERVER` if you installed valid SSL certificates, or `http://1.2.3.4:3000` if you haven't.

3. Create an instance
```
ansible-playbook bootnode.yml
```
Wait till the command completes, extract from logs and write down IP address and AWS InstanceID of the new node.

4. Create/replace file `hosts` with the following content (assuming new node's IP is 1.2.3.4)
```
[bootnode]
1.2.3.4
```

5. Configure the instance
```
ansible-playbook -i hosts site.yml -t bootnode
```

6. When creating first few bootnodes, you need to update `nodes/bootnodes.txt` file in your branch of azure repository, as it will contain enodes of "public" bootnodes. To get enode, ssh to the node and run:
```
curl --data '{"method":"parity_enode","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST localhost:8545
```
then open bootnodes.txt on azure and insert enode on a new line at the end of file

7. When you're done creating as many public bootnodes as necessary, it is recommended to login to each one of them, update local version of `/home/bootnode/bootnodes.txt` and restart parity with
```
systemctl restart oracles-parity
```

8. You may also create a number of "decoy" bootnodes, that will serve to connect nodes of the network, but will not be listed in the public file.

**NOTE**: if you notice that nodes quickly appear and disappear in netstat's dashboard, do the following:
```
ssh root@netstat.ip
systemctl restart oracles-dashboard
```
This should fix the problem from now on. 

### Create a node for Explorer
1. Create a file with a full config for this node:
```
cat group_vars/all.network group_vars/explorer > group_vars/all
```

2. Fill missing values in the end of the file.

3. Create an instance
```
ansible-playbook explorer.yml
```
Wait till the command completes, extract from logs and write down IP address and AWS InstanceID of the new node.

4. Create/replace file `hosts` with the following content (assuming new node's IP is 1.2.3.4)
```
[explorer]
1.2.3.4
```

5. Configure the instance
```
ansible-playbook -i hosts site.yml -t explorer
```

6. If you plan on using Cloudflare/SSL Certificates for the explorer, this is the time to do it. Follow the procedure analogous to netstat server.

### Create MoC's instance and finish the deployment of consensus contracts
1. Create a file with a full config for this node:
```
cat group_vars/all.network group_vars/moc > group_vars/all
```

2. Fill missing values in the end of the file. When setting `MOC_KEYFILE`, paste the entire json content of the keystore file and make sure it's enclosed in single quotes:
```
MOC_KEYFILE: '{"address": ... }'
```
Use `https://netstat.server.com` for `NETSTAT_SERVER` if you installed valid SSL certificates, or `http://1.2.3.4:3000` if you haven't.

3. Create an instance
```
ansible-playbook moc.yml
```
Wait till the command completes, extract from logs and write down IP address and AWS InstanceID of the new node.

4. Create/replace file `hosts` with the following content (assuming new node's IP is 1.2.3.4)
```
[moc]
1.2.3.4
```

5. Configure the instance
```
ansible-playbook -i hosts site.yml -t moc
```

6. After node was created, connect to it via `ssh moc@...` and clone (via https) POA Network Consensus contract repository
```
git clone https://github.com/oraclesorg/poa-network-consensus-contracts.git
git checkout  <correct branch name>
cd poa-network-consensus-contracts
npm install
```
and run the following command to deploy other contracts from the consensus:
```
POA_NETWORK_CONSENSUS_ADDRESS=0xf472e0e43570b9afaab67089615080cf7c20018d MASTER_OF_CEREMONY=<MOC_ADDRESS> ./node_modules/.bin/truffle migrate --reset --network sokol
```
copy and save the output as it contains addresses to which other contracts were deployed.

7. To distribute initial tokens, go to (you are under `moc` user, not `root`!)
```
cd ~/oracles-scripts-owner/distributeTokens
```
Upload csv file with `wallet,tokens` list, then edit `.env` file: replace `FAT_BALANCE` with your MoC address and `FILENAME_CSV_INVESTORS` with the csv file name. Run 
```
node distribute.js
```
to distribute tokens.

8. **To generate initial keys** go to
```
cd ~/oracles-scripts-owner
```
open `config.json` and under "contracts.KeysManager" block replace "addr" with KeysManager contract's address that you obtained while deploying other contracts from the consensus. Don't change ABI unless you've updated contract's code. Then do
```
cd generateInitialKey
node generateInitialKey
```
Script will output initial key's address, password and location of keystore file.

Repeat this step as many times as necessary.

9. Edit `~/node.toml` and comment out the `unlock=["0x...]` line, then restart parity:
```
systemctl restart oracles-parity
```

## Chapter IV - in which MoC prepares other repositories

### DApp - Keys generation  
https://github.com/oraclesorg/oracles-dapps-keys-generation/tree/mainnet

1. in `src/getWeb3.js` change number to `NetworkID`
    switch (netId) {

2. in `src/keysManager.js` change `KEYS_MANAGER_ADDRESS` to the one you obtained when deploying other contracts of consensus

### Repository with scripts for `owner` node
https://github.com/oraclesorg/oracles-scripts-owner/tree/mainnet
1. Update `contracts.KeysManager.addr` in `config.json` to the one you obtained when deploying other contracts of consensus (same thing as you did manually on moc's node).

### Repository with scripts for `validator` node
https://github.com/oraclesorg/oracles-scripts-validator

## Chapter VI - in which MoC gives initial keys to first validators and hopes for the best
For each validator, you will need to provide:
* initial key's address
* initial key's keystore file
* initial key's password
* netstats server's IP
* netstats password
* link to the correct README
* link to the KeysGenerator DApp
