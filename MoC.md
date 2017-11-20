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

### Initial keys
https://github.com/oraclesorg/oracles-initial-keys

### Azure templates
https://github.com/oraclesorg/test-templates

#### What to replace:
1. edit TestTestNet/common.vars and replace branch names where necessary
* `OWNER_ADDRESS`, e.g. `OWNER_ADDRESS="0xdd0bb0e2a1594240fed0c2f2c17c1e9ab4f87126"` - new address of the `owner`.
* `SCRIPTS_BRANCH`, e.g. `SCRIPTS_BRANCH="sokol"` - branch to use in oracles-scripts
* `DAPPS_BRANCH`, e.g. `DAPPS_BRANCH="sokol"` - branch to use in oracles-dapps-*
* `IKEYS_BRANCH`, e.g. `IKEYS_BRANCH="sokol"` - branch to use in oracles-initial-keys
2. you may also wish to update parity and node.js versions for the new network
* `PARITY_INSTALLATION_MODE`, should be either `"BIN"` to install and use binary file directly; or `"DEB"` to `dpkg -i` from package, e.g. `PARITY_INSTALLATION_MODE="BIN"`
* `PARITY_BIN_LOC`, e.g. `PARITY_BIN_LOC="https://transfer.sh/PhhDc/parity"` - location of the binary (used only if `PARITY_INSTALLATION_MODE="BIN"`)
* `PARITY_DEB_LOC`, e.g. `PARITY_DEB_LOC="https://parity-downloads-mirror.parity.io/v1.8.1/x86_64-unknown-linux-gnu/parity_1.8.1_amd64.deb"` - location of the deb package (used only if `PARITY_INSTALLATION_MODE="DEB"`)
* `NODE_SOURCE_DEB`, e.g. `NODE_SROUCE_DEB="https://deb.nodesource.com/setup_6.x"` - location of the node.js package to use
3. when you've done that, you will have to open each of azure templates one by one
* TestTestNet/bootnode/template.json
* TestTestNet/mining-node/template.json
* TestTestNet/netstats-server/template.json
* TestTestNet/owner/template.json

scroll down to `variables` section and change the TEMPLATES_BRANCH value to `NetworkName`, e.g.
```
...
  "variables": {
    "TEMPLATES_BRANCH": "dev-mainnet",
...
```

## Chapter III - in which MoC creates first nodes of the new network

## Chapter IV - in which MoC deploys governance contract

## Chapter V - in which MoC uses all his secret powers to create initial keys

## Chapter VI - in which MoC takes a little break to update links in README
Please update the button in README from master branch
https://github.com/oraclesorg/test-templates

Namely, you need to add _url-encoded_ link to _raw code_ of the validator's node template after https://portal.azure.com/#create/Microsoft.Template/uri/  
You can use https://www.url-encode-decode.com/ to perform encoding.

## Chapter VII - in which MoC gives initial keys to first validators and hopes for the best
