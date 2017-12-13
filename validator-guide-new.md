## Prerequisites

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
1. follow [this instruction](http://docs.ansible.com/ansible/latest/intro_installation.html) to install ansible. For example, you can use `pip` to do it:
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
insert your email address there and strong password. By default, keys will be saved to `~/.ssh/` and named `id_rsa` with your public key being `~/.ssh/id_rsa.pub`.

### 5. aws cli
1. check if you have aws cli installed
```
aws --version
```
if not - install it following [these instructions](http://docs.aws.amazon.com/cli/latest/userguide/installing.html). The simplest way is to use `pip`:
```
pip install awscli --upgrade --user
```

## Configuring AWS
1. register (if you haven't already) and login to the AWS management console https://aws.amazon.com/console/

2. to create credentials for cli, open IAM home https://console.aws.amazon.com/iam/home then click "Add user" pick a username, and check "Programmatic access" for "Access type". Click "Next"

3. you can choose any of the available options, but "Attach existing policies directly" is the simplest one. In the list of policy types check "AmazonEC2FullAccess". Review your account and click "Create user" to proceed.

4. it is very important that you copy "Access Key ID" and "Secret Access Key" without leaving this page, because there is no other way to retrieve "Secret Access Key" later and you will have to start again and create another user.

5. when you've copied and saved your AWS secret keys, next step is to upload your SSH public key. In the top left corner of the page select "Services > EC2". On the left sidebar select "Network & Security" > "Key Pairs". Click "Import Key Pair". Browse your filesystem for the public key. You can give a name to this keypair, otherwise base name of the file will be used (by default `id_rsa`).

6. configure aws cli:
```
aws configure
```
provide your credentials, choose region for your account (e.g. `us-east-2`) and output format (`json` is recommended).

7. check that keypair was correctly imported:
```
aws ec2 describe-key-pairs
```
you should see your keypair name in the list.

## Download and configure playbook
1. clone repository with ansible playbooks
```
git clone https://github.com/oraclesorg/deployment-playbooks.git
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

4. choose subnet:
```
aws ec2 describe-subnets
```

5. other parameters


## Deployment

### Create instance

### Configure instance

### Close external access
