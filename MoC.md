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

## Chapter II - in which MoC forks a lot of repos and replaces some text

There are quite a few repositories that are used to run the network. You will need to fork them and later update parameters.
Please be consistent with naming of branches and use `NetworkName`.

### DApps
1. Keys generation  
https://github.com/poanetwork/poa-dapps-keys-generation
2. Voting  
https://github.com/poanetwork/poa-dapps-voting
3. Validators list  
https://github.com/poanetwork/poa-dapps-validators

#### What to replace:
1. in each dapp go to `assets/javascripts/config.json` and change networkID to new network's `NetworkID`

### Contract
https://github.com/poanetwork/poa-network-consensus-contracts

#### What to replace:
1. open `src/Owned.sol` file and update `owner` variable in `function Owned()`
2. clone the repository
3. make sure you have the latest version of `truffle` installed:
```
sudo npm -g install truffle
```
3. do `npm install` in the repository
4. open a new tab and run
```
make testrpc
```
5. in the first tab run
```
truffle compile
```
6. when compilation is completed go to `build/contracts` folder, you will later need `bytecode` and `abi` properties from the following `json` files:
```
BallotsManager.json
BallotsStorage.json
KeysManager.json
KeysStorage.json
ValidatorsManager.json
ValidatorsStorage.json
```

### Repository with `chain.json`
https://github.com/poanetwork/poa-chain-spec

#### What to replace:
1. in `spec.json` update address for the `owner` in `accounts` section (so that `owner` has nonzero balance)
2. place new contract's abi code in the following accounts' `constructor` fields:
```
0xf472e0e43570b9afaab67089615080cf7c20018d - ValidatorsStorage
0xbbeeea48d60b8c24eaefa334a503509e23d5e515 - ValidatorsManager
0xeb1352fa30033da7f2a7b50a033ed47ef4b178a6 - KeysStorage
0x8c9b4b504e6ffe7bc2f2811abc1fe0a2ef87fa5b - KeysManager
0xdebe80f4800a23db154d023190d0658c1a6c033a - BallotsStorage
0xfd3c58bc0dc90c4d09b79e99a7ef6318e2342100 - BallotsManager
```
3. change `params.networkID` to new network's `NetworkID` in hex format.

### Repository with scripts for `owner` node
https://github.com/poanetwork/poa-scripts-moc

#### What to replace:
Unless you updated contract's code besides changing `owner`, you don't need to update anything, because only ABI is used in this repo.

### Repository with scripts for `validator` node
https://github.com/poanetwork/poa-scripts-validator

#### What to replace:
Unless you updated contract's code besides changing `owner`, you don't need to update anything, because only ABI is used in this repo.

### Azure templates
https://github.com/poanetwork/deployment-azure

#### What to replace:
0. open `nodes/bootnodes.txt` and remove it's entire content
1. when you've done that, you will have to open each of azure templates one by one
* `nodes/bootnode/template.json`
* `nodes/mining-node/template.json`
* `nodes/netstats-server/template.json`
* `nodes/owner/template.json`

scroll down to `variables` section and change the `TEMPLATES_BRANCH` value to `NetworkName`, e.g.
```
...
  "variables": {
    "TEMPLATES_BRANCH": "dev-mainnet",
...
```
If you forked the original repo to your account, also replace the `MAIN_REPO_FETCH` value with your account name, e.g.
```
...
  "variables": {
    "TEMPLATES_BRANCH": "dev-mainnet",
    "MAIN_REPO_FETCH": "poanetwork",
...
```

2. edit `nodes/common.vars` and replace branch names of repositories where necessary

3. update links of buttons in README.md  
Namely, in each button you need to replace _url encoded_ link to _raw code_ (https://raw.githubusercontent.com/...) of the node's template.json after https://portal.azure.com/#create/Microsoft.Template/uri/  
You can use https://www.url-encode-decode.com/ to perform url encoding.

This is an example:
```
[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fpoanetwork%2Fdeployment-azure%2Fdev-mainnet%2Fnodes%2Fmining-node%2Ftemplate.json)
```

### Ansible playbook
https://github.com/poanetwork/deployment-playbooks

#### What to replace:
Open `group_vars/all.network` and replace with `NetworkName`
* `TEMPLATES_BRANCH`
* `SCRIPTS_OWNER_BRANCH` 
* `SCRIPTS_VALIDATOR_BRANCH`
* `GENESIS_BRANCH`
* `OWNER_ADDRESS`

If you forked the original repo to your account, also replace the `MAIN_REPO_FETCH` value with your account name.

## Chapter III - in which MoC creates first nodes of the new network
There is a "chicken and egg" problem here, because you first need to create bootnode and netstats server, however bootnode needs to send statistics to netstats and netstats needs to connect to a bootnode. It seems to be easier to start from netstats.

### (III.a) using Azure templates
0. open README
1. click on "Netstats server" button. Fill all required fields, it is recommended to use a bigger `vmSize` for netstats. Set a strong `netstats password`. Wait till the node is created. After that, you should be able to access dashboard on http://1.2.3.4:3000 and access explorer on http://1.2.3.4:4000 . There will be nothing there yet though. **Write down netstats ip address**.
2. click on "Bootnode" button. Fill all required fields, it is recommended to use a bigger `vmSize` for bootnode too. Wait till the node is created. Log in to the node, run this script to get bootnode's enode:
```
curl --data '{"method":"parity_enode","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST localhost:8545
```
copy it and add to `nodes/bootnodes.txt` on a separate line

3. log out from bootnode and log back in to netstats server. Copy content of bootnodes.txt from repo or run this script in home folder to re-download it:
```
curl -sLO https://raw.githubusercontent.com/$MAIN_REPO_FETCH/deployment-azure/$TEMPLATES_BRANCH/nodes/bootnodes.txt
```
then restart parity:
```
sudo systemctl restart oracles-parity
```
You may need to restart chain explorer, netstats daemon and dashboard, as they sometimes disconnect when parity disconnects
```
sudo systemctl restart oracles-dashboard
pm2 restart all
```

4. go back to README and click on "Owner" button. Fill all required fields, use keystore file and password that you generated in the first chapter.

### (III.b) using Ansible playbooks
0. make sure you have installed `python`, `ansible`, `aws` cli and configured aws access keys.  
Create a file `files/admins.pub` and paste your public ssh key there `ssh-rsa AAA...`. Then create copies of this file for all roles:
```
cp files/admins.pub files/ssh_netstat.pub
cp files/admins.pub files/ssh_bootnode.pub
cp files/admins.pub files/ssh_explorer.pub
cp files/admins.pub files/ssh_owner.pub
```

1. To create netstat, copy network-wide settings from `group_vars/all.network` and extend them with role-specific settings
```
cat group_vars/all.network group_vars/netstat.example > group_vars/all
```
then edit `group_vars/all` and fill remaining fields.

Run ansible to create and launch EC2 instance
```
ansible-playbook netstat.yml
```
when the instance is launched, **note it's ip address** (let's assume ip is 1.2.3.4), then create file `hosts` with the following content
```
[netstat]
1.2.3.4
```
and run ansible again to configure the instance
```
ansible-playbook -i hosts site.yml -t netstat
```
If you see an error that host is unavailable over SSH, wait a few minutes and try again, as it probably has not yet completed rebooting.

After process is completed, if you want to install custom ssl certificates, ssh to the host as root, stop nginx:
```
service nginx stop
```
switch to the nginx folder, make backup copies of current certificate and key, leave `dhparams` in place:
```
cd /etc/nginx
mkdir -p ssl.orig
mv ssl/server.crt ssl/server.key ssl.orig/
```
put your `server.crt` and `server.key` into the `ssl` folder and start nginx:
```
service nginx start
```
check that it's running:
```
ps aux | grep nginx
```
if not, check logs for errors.  
After certificates are installed, open `group_vars/all.network` and update `NETSTAT_SERVER` with the correct https://... url, commit your change to `group_vars/all.network` and push it to github. **MAKE SURE you didn't commit group_vars/all and didn't accidently add your aws keys, owner keys or validator keys** 

2. to create bootnode remove current `group_vars/all` and create a new one
```
rm group_vars/all
cat group_vars/all.network group_vars/bootnode.example > group_vars/all
```
then edit `group_vars/all` and fill remaining fields.

Run ansible to create and launch EC2 instance
```
ansible-playbook bootnode.yml
```
when the instance is launched, note it's ip address (let's assume ip is 1.2.3.4), then clear file `hosts` and add following lines
```
[bootnode]
1.2.3.4
```
and run ansible again to configure the instance
```
ansible-playbook -i hosts site.yml -t bootnode
```
If you see an error that host is unavailable over SSH, wait a few minutes and try again, as it probably has not yet completed rebooting.

Log in to the node, run this script to get bootnode's enode:
```
curl --data '{"method":"parity_enode","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST localhost:8545
```
copy it and add to `https://github.com/${MAIN_REPO_FETCH}/deployment-azure/blob/${TEMPLATES_BRANCH}/nodes/bootnodes.txt` on a separate line.

If you want to replace ssl certificates on bootnode, follow the procedure from step 1.

If you see that bootnode periodically becomes offline on the netstat dashboard, ssh to netstat server over ssh and restart oracles-dashboard:
```
systemctl restart oracles-dashboard
```
that should fix the issue.

3. to create a node with chain explorer remove current `group_vars/all` and create a new one
```
rm group_vars/all
cat group_vars/all.network group_vars/explorer.example > group_vars/all
```
then edit `group_vars/all` and fill remaining fields.

Run ansible to create and launch EC2 instance
```
ansible-playbook explorer.yml
```
when the instance is launched, note it's ip address (let's assume ip is 1.2.3.4), then clear file `hosts` and add following lines
```
[explorer]
1.2.3.4
```
and run ansible again to configure the instance
```
ansible-playbook -i hosts site.yml -t explorer
```
If you see an error that host is unavailable over SSH, wait a few minutes and try again, as it probably has not yet completed rebooting.

If you want to replace ssl certificates on explorer, follow the procedure from step 1.

4. to create a node for master of ceremony remove current `group_vars/all` and create a new one
```
rm group_vars/all
cat group_vars/all.network group_vars/owner.example > group_vars/all
```
then edit `group_vars/all` and fill remaining fields.

Run ansible to create and launch EC2 instance
```
ansible-playbook owner.yml
```
when the instance is launched, note it's ip address (let's assume ip is 1.2.3.4), then clear file `hosts` and add following lines
```
[owner]
1.2.3.4
```
and run ansible again to configure the instance
```
ansible-playbook -i hosts site.yml -t owner
```
If you see an error that host is unavailable over SSH, wait a few minutes and try again, as it probably has not yet completed rebooting.

## Chapter IV - in which MoC joins governance contracts
Log in to owner's node, go to `poa-scripts-moc/joinContracts` folder and run
```
node joinContracts.js
```
you should receive output that contracts were joined.

## Chapter V - in which MoC uses all his secret powers to create initial keys
Log in to owner's node, open `node.toml` and temporarily add line in the `[account]` section
```
unlock=["0x..."] # owner's address
```
restart parity
```
sudo systemctl restart oracles-parity
```

switch to `poa-scripts-moc/generateInitialKey` folder and run the script to generate initial key
```
node generateInitialKey.js
```
the script will output initial key's address, password and location of keystore file.

Repeat this procedure as many times as necessary.

Remove or comment `unlock=...` line from `node.toml` and restart parity again.

## Chapter VI - in which MoC takes a little break to update links in README
If new network is of main-net variety, for user's convenience please update the link in the button in README from master branch
https://github.com/poanetwork/deployment-azure
to point to the mining-node's template

## Chapter VII - in which MoC gives initial keys to first validators and hopes for the best
For each validator, you will need to provide:
* initial key's address
* initial key's keystore file
* initial key's password
* netstats server's IP
* netstats password
* link to the correct README
* link to the KeysGenerator DApp
