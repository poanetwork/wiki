## Exchange your initial keys for mining, payout and voting keys
1. Start Chrome
2. Connect to the network in MetaMask - click on the network name in the top left corner of plugin's window and in the dropdown list select "Custom RPC", enter URL that was provided to you by the Master of Ceremony (For Core network: [https://core.poa.network](https://core.poa.network) for Sokol testnet [https://sokol.poa.network](https://sokol.poa.network)). Wait till the MetaMask connects to the network
3. Open Keys DApp: for Core network: [https://core-ceremony.poa.network/](https://core-ceremony.poa.network/), for Sokol testnet: [https://sokol-ceremony.poa.network/](https://sokol-ceremony.poa.network/)
4. Upload your initial key to MetaMask
5. Click "Generate keys", confirm transaction.
6. **Be sure to copy address, password and download keystore file for each key (mining, payout, voting) without closing browser's tab**. There is no way to get this data once you close the tab. Keep it in a safe place.

## How to setup validator's node on hosting other than AWS

0. make sure you have Python 2 (versions 2.6 or 2.7) or Python 3 (versions 3.5 and higher) installed on your local machine (Windows isn't supported for the control machine) and Ansible v2.3+

1. setup an Ubuntu 16.04 server

2. to run playbook you will need a user on the server, who can execute `sudo` wihout password and who can be logged in via SSH public key. By default it is assumed that this user is called `ubuntu`. If you already have a user with different name who satisfies these requirements, at the top of `site.yml` in `-hosts: all` section change line `user: ubuntu` to the name you have
```
---
- hosts: all
  user: another-user
  become: True
...
```
_NOTE_: playbook will additionally create a new unprivileged user named `validator` and add your ssh public key to `root` account.

3. clone repository with ansible playbooks and checkout branch with the network name you want to join (e.g. `core` for mainnet and `sokol` for testnet)

```
git clone https://github.com/poanetwork/deployment-playbooks.git
cd deployment-playbooks
# for core mainnet
git checkout core
# OR for sokol testnet
git checkout sokol
# check that you ended up on a correct branch (look where the `*` is)
git branch
```

4. put ssh public keys (in format "ssh AAA...") that need access to the server to both files
```
files/admins.pub
files/ssh_validator.pub
```
(one key per line)

5. create configuration file
```
cat group_vars/all.network group_vars/validator.example > group_vars/all
```

6. edit the `group_vars/all` file and comment out parameters corresponding to aws:
```
#access_key
#secret_key
#awskeypair_name
#vpc_subnet_id
```

7. set values for the following parameters in `group_vars/all`:
* `NODE_FULLNAME` - your real name (will be visible to other mebers of the network)
* `NODE_ADMIN_EMAIL` - your public email address (will be visible to other members of the network)
* `MINING_KEYFILE` - insert content of your mining keystore json file. Resulting value should be enclosed in single quotes and look similar to this: `MINING_KEYFILE: '{"address":"..."}'`
* `MINING_ADDRESS` - insert your mining key address, e.g. `MINING_ADDRESS: "0x..."`
* `MINING_KEYPASS` - insert your mining key password

8. set values given to you by Master of Ceremony for the following parameters in `group_vars/all`:
* `NETSTATS_SERVER`
* `NETSTATS_SECRET`

9. set the following options as follows in `group_vars/all`:
```
allow_validator_ssh: true
allow_validator_p2p: true
associate_validator_elastic_ip: false
```
_Double check that_ `allow_validator_ssh` _is_ `true` _otherwise you won't be able to connect to the node_.

10. create file `hosts` with the server's ip address (e.g. 192.0.2.1):
```
[validator]
192.0.2.1
```

11. run ansible playbook
```
ansible-playbook -i hosts site.yml
```

12. open `NETSTATS_SERVER` url in the browser and check that the node named `NODE_FULLNAME` appeared in the list

### Obtaining enode uri for Master of Ceremony
Login to the node and get enode from parity logs:
```
ssh root@192.0.2.1
grep enode /home/bootnode/logs/parity.log
```
copy `enode` uri and send it to Master of Ceremony. If this line is not found, restart parity
```
systemctl restart poa-parity
```
and try again. If `enode` uri is still not found, use the commands below to restart all services.

_NOTE_ if after parity restart you notice that on `NETSTATS_SERVER` url your node starts to fall behind other nodes (block number is less than on other nodes), try to restart statistics service (assuming you are connected as `root`):
```
su bootnode
pm2 restart all
```
after that refresh `NETSTATS_SERVER` url and check again your node's block number. If your node is still not active or missing `enode`, log in to root account and reboot the OS. 
```
su 
shutdown -r now
```
