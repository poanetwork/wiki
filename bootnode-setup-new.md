0. make sure you have Python 2 (versions 2.6 or 2.7) or Python 3 (versions 3.5 and higher) installed on your local machine (Windows isn’t supported for the control machine) and Ansible v2.3+

1. setup an Ubuntu 16.04 server

2. create a user `ubuntu` that can execute `sudo` without password

3. download playbook
```
git clone https://github.com/oraclesorg/deployment-playbooks.git
git checkout core
```

4. create configuration file
```
cp group_vars/all.network group_vars.bootnode.example > group_vars/all
```

5. edit the `group_vars/all` file and comment out lines corresponding to aws:
```
#access_key
#secret_key
#awskeypair_name
#vpc_subnet_id
```

6. set values for the following parameters in `group_vars/all`:
* `NODE_FULLNAME`
* `NODE_ADMIN_EMAIL`
* `NETSTATS_SERVER`
* `NETSTATS_SECRET`

7. comment out this line in `site.yml` in `hosts: bootnode` section
```
#- include: bootnode-access.yml
```

8. create file `hosts` with the server's ip address (e.g. 192.0.2.1):
```
[bootnode]
192.0.2.1
```

9. run ansible playbook
```
ansible-playbook -i hosts site.yml -l 192.0.2.1
```

10. open `NETSTATS_SERVER` url in the browser and check that the node named `NODE_FULLNAME` appeared in the list

11. login to the node under `root` and run the following command:
```
curl --data '{"method":"parity_enode","params":[],"id":1,"jsonrpc":"2.0"}' -H "Content-Type: application/json" -X POST localhost:8545
```
copy `enode` uri and send it to Master of Ceremony

12. restrict incoming connections to ports `22` and `30303` only
