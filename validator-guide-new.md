## Exchange your initial keys for mining, payout and voting keys
1. Start Chrome
2. Connect to the network in MetaMask - click on the network name in the top left corner of plugin's window and in the dropdown list select "Custom RPC", enter URL that was provided to you by the Master of Ceremony. Wait till the MetaMask connects to the network
3. Open DApp http://example.com
4. Click "Generate keys", confirm transaction.
5. **Be sure to copy address, password and download keystore file for each key (mining, payout, voting) without closing browser's tab**. There is no way to get this data once you close the tab. Keep it in a safe place.

## Validator's node Setup prerequisites

### 1. git
1. check that you have git installed
```
git --version
```
if not - install it following instructions [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git)

### 2. python & pip
1. check that you have python 2 version >= 2.6.5 or python 3 version >= 3.3 installed
```
python --version
```
if not - install it choosing apropriate binary from [here](https://www.python.org/downloads/)

2. check if you have `pip` python package manager install
```
pip --version
```
if not - install it following instructions from [here](https://pip.pypa.io/en/stable/installing/). Basically, you need to download this script and save it on your computer https://bootstrap.pypa.io/get-pip.py then run
```
python get-pip.py
```

### 3. ansible
1. follow [this guide](http://docs.ansible.com/ansible/latest/intro_installation.html) to install ansible. For example, you can use `pip` to do it:
```
sudo pip install ansible
```

2. use `pip` to install the following packages:
```
sudo pip install boto
sudo pip install boto3
```

### 4. SSH keys
1. check if you already have a keypair:
```
ls -la ~/.ssh
```
if you get error that directory does not exist or the directory is empty, you need to follow the instructions below. If you already have key pair, you can skip this section.

2. generate ssh key-pair
```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
insert your email address there and a strong password. By default, keys will be saved to `~/.ssh/` and named `id_rsa` with your public key being `~/.ssh/id_rsa.pub`.

### 5. aws cli
1. check if you have aws cli installed
```
aws --version
```
if not - install it following [these instructions](http://docs.aws.amazon.com/cli/latest/userguide/installing.html). The simplest way is to use `pip`:
```
pip install awscli --upgrade --user
```
Mac systems with homebrew installed:
```
brew install awscli
```

## Configuring AWS
1. Register (if you haven't already) and login to the AWS management console: https://aws.amazon.com/console/

2. to create credentials for cli, open IAM home https://console.aws.amazon.com/iam/home, select "Users" on the left hand side mav bar and then click "Add user".  Pick a username, and check "Programmatic access" for "Access type". Click "Next:Permissions"

3. you can choose any of the available options, but "Attach existing policies directly" is the simplest one. In the list of policy types search for and then check "AmazonEC2FullAccess". Click "Next:Review".  Review your account and click "Create user" to proceed.

4. it is very important that you copy "Access Key ID" and "Secret Access Key" without leaving this page, because there is no other way to retrieve "Secret Access Key" later and you will have to start again and create another user. After copying this important information, select "Close". 

5. after you've copied and saved your AWS secret keys, the next step is to upload your SSH public key. In the top left corner of the page select "Services -> EC2". On the left sidebar select "Network & Security" -> "Key Pairs". Click "Import Key Pair". Give a name to this keypair, otherwise base name of the file will be used (by default `id_rsa`). Browse your filesystem for the public key, or copy/paste:
```
pbcopy < ~/.ssh/id_rsa.pub
```
This will copy your public key into your clipboard and can then be pasted.

6. configure aws cli:
```
aws configure
```
provide your credentials (Access Key ID and Secret Access Key) from earlier. Choose a region for your account (e.g. `us-east-2`) and output format (`json` is recommended).

7. check that keypair was correctly imported:
```
aws ec2 describe-key-pairs
```
you should see your keypair name in the list.

## Download and configure playbook
You may need to add your github info, if you haven't already.  This may require the creation of a new "Personal Access Token".

1. clone repository with ansible playbooks
```
git clone https://github.com/oraclesorg/deployment-playbooks.git
cd deployment-playbooks
```

2. prepare files with ssh keys
```
cat ~/.ssh/id_rsa.pub > files/admins.pub
cp files/admins.pub files/ssh_validator.pub
```

3. create file with configuration settings:
```
cat group_vars/all.network group_vars/validator.example > group_vars/all
```

4. to choose subnet run the following command
```
aws ec2 describe-subnets
```
select any subnet with "State": "available" and non-zero "AvailableIpAddressCount". You need to copy/save "SubnetId" of this subnet for later use.

5. open `group_vars/all` and edit the following configuration options:
```
nano group_vars/all
```

Make the following edits:

* `access_key` - your AWS "Access Key ID"
* `secret_key` - your AWS "Secret Access Key"
* `awskeypair_name` - name of ssh keypair you uploaded on AWS (by default `id_rsa`)
* `vpc_subnet_id` - insert "SubnetId" that you chose. The next line should look like this:
```
vpc_subnet_id: "subnet-..."
```
* `NODE_FULLNAME` - enter your full name (this will be visible to other members of the network)
* `NODE_ADMIN_EMAIL` - enter your public email (this will be visible to other members of the network)
* `NETSTATS_SERVER` - this should be a url provided to you by the Master of Ceremony
* `NETSTATS_SECRET` - this should be a secret code provided to you by the Master of Ceremony
* `MINING_KEYFILE` - insert _content_ of your mining keystore file. Resulting value should be enclosed in single quotes and look similar to this:
```
MINING_KEYFILE: '{"address":"..."}'
```
* `MINING_ADDRESS` - insert your mining key address, e.g.
```
MINING_ADDRESS: "0x..."
```
* `MINING_KEYPASS` - insert your mining key's passphrase

6. examine values in `image` and `region` properties. If your AWS region doesn't match the one in `region` you need to replace `region` with the correct one and select image from this list https://cloud-images.ubuntu.com/locator/ec2/ 

Open this page, scroll down, choose your region from the first ("Zone") dropdown list, choose `xenial` from the second ("Name") dropdown list and `hvm:ebs-ssd` from the fifth ("Instance type"). This should limit you to a single option, copy value from "AMI-ID" column and paste it in `image` property.

7. you may also choose a different value for the `validator_instance_type`. For `region: "us-east-2"` we recommend using `m4.xlarge`. Confirm your option of the types of instances available in your region,via: https://aws.amazon.com/ec2/pricing/on-demand/

## Deployment
### Create instance
1. with all options configured, you first need to create an instance: (you should still be in: ~/deployment-playbooks)
```
ansible-playbook validator.yml
```
this script will ask you for your SSH key passphrase unless you didn't set a passphrase or you entered it recently.

2. after this process is complete, examine script's output and write down IP (e.g. `192.0.2.1`) address and AWS InstanceID (e.g. `i-0123456789abcdef0`) for later use.

### Configure instance
1. create file `hosts` with the following content (assuming IP address is `192.0.2.1`)
```
[validator]
192.0.2.1
```

2. run this script to configure the instance
```
ansible-playbook -i hosts site.yml -t validator
```
if you get an error that host cannot be reached over SSH, please wait a minute and start again. This error may appear because instance is rebooted after creation, and this may take some time to complete.

3. open the url for `NETSTAT_SERVER` and check if your node appeared in the list

### Close external access
1. if all worked fine and your node appeared on the list with the same block number as other nodes, you need to close access to your node. Edit `group_vars/all` and set these options to `false`:
```
allow_validator_ssh: false
allow_validator_p2p: false
```
then run
```
ansible-playbook validator-access.yml
```

2. if you later need to re-enable access, change necessary options to `true` and run the script again. E.g. to re-enable ssh access to your node, set `allow_validator_ssh: true`.

NOTE: this script applies simultaneously to all your instances with security group named `validator-security`. This note is relevant only if you have several instances of validator nodes running in the same region.

### Remove instance
In case you want to remove your instance

a. do it via AWS GUI: open AWS management console https://console.aws.amazon.com/ec2/v2/home#Instances check the instance you want to remove, click Actions > Instance State > Terminate.

b. do it via aws cli: get AWS Instance ID (the one you saved previously, or you can look it up in AWS management console) and run
```
aws ec2 terminate-instances --instance-ids i-0123456789abcdef0
```
(replace `i-0123456789abcdef0` with your actual AWS InstanceID).

NOTE: this operation is irrevertable and if need be, you'll have to create a new instance from scratch.
