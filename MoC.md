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
1.
2.
3.

### Payout script (also containig `chain.json`)
https://github.com/oraclesorg/oracles-scripts

#### What to replace:
1. in `spec.json` update address for the `owner` in `accounts` section (so that `owner` has nonzero balance)
2. if you changed contract's code, in `scripts/config.json` update `Ethereum.contracts.Oracles.abi`.

### Initial keys
https://github.com/oraclesorg/oracles-initial-keys
#### What to replace:
1. in `config.json` replace `Ethereum.live.account` with address of the new `owner`
2. if you updated contract's code, also replace `Ethereum.contracts.Oracles.bin` and `.abi`.

### Azure templates
https://github.com/oraclesorg/deployment-azure

#### What to replace:
0. open `nodes/bootnodes.txt` and remove it's entire content
1. edit `nodes/common.vars` and replace branch names where necessary
* `OWNER_ADDRESS`, e.g. `OWNER_ADDRESS="0xdd0bb0e2a1594240fed0c2f2c17c1e9ab4f87126"` - new address of the `owner`.
* `SCRIPTS_BRANCH`, e.g. `SCRIPTS_BRANCH="sokol"` - branch to use in oracles-scripts
* `DAPPS_BRANCH`, e.g. `DAPPS_BRANCH="master"` - branch to use in oracles-dapps-*
* `IKEYS_BRANCH`, e.g. `IKEYS_BRANCH="master"` - branch to use in oracles-initial-keys
2. you may also wish to update parity and node.js versions for the new network
* `PARITY_INSTALLATION_MODE`, should be either `"BIN"` to install and use binary file directly; or `"DEB"` to `dpkg -i` from package, e.g. `PARITY_INSTALLATION_MODE="BIN"`
* `PARITY_BIN_LOC`, e.g. `PARITY_BIN_LOC="https://transfer.sh/PhhDc/parity"` - location of the binary (used only if `PARITY_INSTALLATION_MODE="BIN"`)
* `PARITY_DEB_LOC`, e.g. `PARITY_DEB_LOC="https://parity-downloads-mirror.parity.io/v1.8.1/x86_64-unknown-linux-gnu/parity_1.8.1_amd64.deb"` - location of the deb package (used only if `PARITY_INSTALLATION_MODE="DEB"`)
* `NODE_SOURCE_DEB`, e.g. `NODE_SOURCE_DEB="https://deb.nodesource.com/setup_6.x"` - location of the node.js package to use
3. when you've done that, you will have to open each of azure templates one by one
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
4. update links of buttons in README  
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
