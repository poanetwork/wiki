## How to setup a bootnode not on AWS.

0. make sure you have Python 2 (versions 2.6 or 2.7) or Python 3 (versions 3.5 and higher) installed on your local machine (Windows isn't supported for the control machine) and Ansible v2.3+

1. setup an Ubuntu 16.04 server

2. to run playbook you will need a user who can execute `sudo` wihout password and who can be logged in via SSH public key. By default it is assumed that this user is called `ubuntu`. If you already have a user with different name who satisfies these requirements, at the top of `site.yml` in `-hosts: all` section change line `user: ubuntu` to the name you have
```
---
- hosts: all
  user: another-user
  become: True
...
```
_NOTE_: playbook will additionally create a new unprivileged user named `bootnode`.

3. download playbook (substitute `core` with another network's name if you're not connecting to the main network)
```
git clone https://github.com/oraclesorg/deployment-playbooks.git
cd deployment-playbooks
git checkout core
```

4. put ssh public keys (in format "ssh AAA...") that need access to the server to both files
```
files/admins.pub
files/ssh_bootnode.pub
```
(one key per line)

5. create configuration file
```
cat group_vars/all.network group_vars/bootnode.example > group_vars/all
```

6. edit the `group_vars/all` file and comment out parameters corresponding to aws:
```
#access_key
#secret_key
#awskeypair_name
#vpc_subnet_id
```

7. set values given to you by Master of Ceremony for the following parameters in `group_vars/all`:
* `NODE_FULLNAME`
* `NODE_ADMIN_EMAIL`
* `NETSTATS_SERVER`
* `NETSTATS_SECRET`

8. set the following options as follows:
```
allow_bootnode_ssh: true
allow_bootnode_p2p: true
allow_bootnode_rpc: false
associate_bootnode_elastic_ip: false
```
_Double check that_ `allow_bootnode_ssh` _is_ `true` _otherwise you won't be able to connect to the node_.

9. create file `hosts` with the server's ip address (e.g. 192.0.2.1):
```
[bootnode]
192.0.2.1
```

10. run ansible playbook
```
ansible-playbook -i hosts site.yml -l 192.0.2.1
```

11. open `NETSTATS_SERVER` url in the browser and check that the node named `NODE_FULLNAME` appeared in the list

12. login to the node and get enode from parity logs:
```
ssh root@192.0.2.1
grep enode /home/bootnode/logs/parity.log
```
copy `enode` uri and send it to Master of Ceremony. If this line is not found, restart parity
```
systemctl restart poa-parity
```
and try again.
