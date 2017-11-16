# ~~not-so~~ Easy steps to create a new network
(for the Master of Ceremony)

Prerequisites:
1. Plenty of time
2. Patience
3. New network's `NetworkID`
4. New network's `CodeName`

## Chapter I - in which MoC creates his secret keys and never shows them to others

You will need to generate ethereum address and keystore file for the `owner`.  
One way is to download `etherwallet-v*.*.*.*.zip` archive of the latest release from myetherwallet github repo https://github.com/kvhnuke/etherwallet/releases/
then unplug your computer from the Internet, extract zip archive and open `index.html` in your browser.
Please be sure to use strong password, download keystore file `UTC--*--*`, your private key and keep them in a safe place.

## Chapter II - in which MoC forks a lot of repos and replaces some text

There are quite a few repositories that are used to run the network. You will need to fork them and later update parameters.
Please be consistent with naming of branches and use `CodeName`.

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

### Azure templates
https://github.com/oraclesorg/test-templates

### Payout script (also containig `chain.json`)
https://github.com/oraclesorg/oracles-scripts

### Initial keys
https://github.com/oraclesorg/oracles-initial-keys

## Chapter III - in which MoC creates first nodes of the new network

## Chapter IV - in which MoC deploys governance contract

## Chapter V - in which MoC uses all his secret powers to create initial keys

## Chapter VI - in which MoC takes a little break to update links in README

## Chapter VII - in which MoC gives initial keys to first validators and hopes for the best
