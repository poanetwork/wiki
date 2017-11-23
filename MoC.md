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
https://github.com/oraclesorg/oracles-dapps-keys-generation
2. Voting  
https://github.com/oraclesorg/oracles-dapps-voting
3. Validators list  
https://github.com/oraclesorg/oracles-dapps-validators

### Contract
https://github.com/oraclesorg/oracles-contract

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
6. when compilation is completed go to `build/contracts` folder

### Repository with `chain.json`
https://github.com/oraclesorg/oracles-chain-spec

#### What to replace:
1. in `spec.json` update address for the `owner` in `accounts` section (so that `owner` has nonzero balance)
2. place new contract's code in the following accounts' `constructor` fields:
```
0xf472e0e43570b9afaab67089615080cf7c20018d
0xbbeeea48d60b8c24eaefa334a503509e23d5e515
0xeb1352fa30033da7f2a7b50a033ed47ef4b178a6
0x8c9b4b504e6ffe7bc2f2811abc1fe0a2ef87fa5b
0xdebe80f4800a23db154d023190d0658c1a6c033a
0xfd3c58bc0dc90c4d09b79e99a7ef6318e2342100
```

### Repository with scripts for `owner` node
https://github.com/oraclesorg/oracles-scripts-owner

### Repository with scripts for `validator` node
https://github.com/oraclesorg/oracles-scripts-validator

### Azure templates
https://github.com/oraclesorg/deployment-azure

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
If you forked the original repo to your account, also replace the `MAIN_REPO_FETCH` value to your account name, e.g.
```
...
  "variables": {
    "TEMPLATES_BRANCH": "dev-mainnet",
    "MAIN_REPO_FETCH": "oraclesorg",
...
```

2. edit `nodes/common.vars` and replace branch names of repositories where necessary

3. update links of buttons in README.md  
Namely, in each button you need to replace _url encoded_ link to _raw code_ (https://raw.githubusercontent.com/...) of the node's template.json after https://portal.azure.com/#create/Microsoft.Template/uri/  
You can use https://www.url-encode-decode.com/ to perform url encoding.

This is an example:
```
[![Deploy to Azure](http://azuredeploy.net/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Foraclesorg%2Fdeployment-azure%2Fdev-mainnet%2Fnodes%2Fmining-node%2Ftemplate.json)
```

## Chapter III - in which MoC creates first nodes of the new network
There is a "chicken and egg" problem here, because you first need to create bootnode and netstats server, however bootnode needs to send statistics to netstats and netstats needs to connect to a bootnode. It seems to be easier to start from netstats.

0. open README
1. click on "Netstats server" button. Fill all required fields, it is recommended to use a bigger `vmSize` for netstats. Set a strong `netstats password`. Wait till the node is created. After that, you should be able to access dashboard on http://1.2.3.4:3000 and access explorer on http://1.2.3.4:4000 . There will be nothing there yet though. **Write down netstats ip address**.
2. click on "Bootnode" button. Fill all required fields, it is recommended to use a bigger `vmSize` for bootnode too. Wait till the node is created. Log in to the node, run this script to get bootnode's enode:
```
curl --data '{"method":"parity_enode","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST localhost:8545
```
copy it and add to `nodes/bootnodes.txt` on a separate line

3. log out from bootnode and log back in to netstats server. Run this script in home folder to re-download bootnodes.txt:
```
curl -sLO https://github.com/oraclesorg/<***** Branch name *****>/TestTestNet/bootnodes.txt
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

## Chapter IV - in which MoC deploys governance contract

## Chapter V - in which MoC uses all his secret powers to create initial keys
Log in to owner's node, open `node.toml` and temporarily add line in the `[account]` section
```
unlock=["0x..."] # owner's address
```
restart parity
```
sudo systemctl restart oracles-parity
```

switch to `oracles-initial-keys` folder and run the script to generate initial key
```
node generateInitialKey.js
```
the script will output initial key's address, password and location of keystore file.

Repeat this procedure as many times as necessary.

Remove `unlock=...` line from `node.toml` and restart parity again.

## Chapter VI - in which MoC takes a little break to update links in README
If new network is of main-net variety, for user's convenience please update the link in the button in README from master branch
https://github.com/oraclesorg/test-templates
to point to the mining-node's template

## Chapter VII - in which MoC gives initial keys to first validators and hopes for the best
For each validator, you will need to provide:
* initial key's address
* initial key's keystore file
* initial key's password
* netstats server's IP
* netstats password
* link to the correct template's branch
