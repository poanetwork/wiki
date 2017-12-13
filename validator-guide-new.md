## Prerequisites

### 1. (optional) git
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
if not - install it following instructions from [here](https://pip.pypa.io/en/stable/installing/). Basically, you need to download this script and save it on your drive https://bootstrap.pypa.io/get-pip.py then run
```
python get-pip.py
```

### 3. aws cli
1. check if you have aws cli installed
```
aws --version
```
if not - install it following [these instructions](http://docs.aws.amazon.com/cli/latest/userguide/installing.html). The simplest way is to use `pip`:
```
pip install awscli --upgrade --user
```

## Generating SSH keys
1. check if you already have a keypair:
```
ls -la ~/.ssh
```
if you get error that directory does not exist or the directory is empty, you need to follow the instructions below. If you already have key pair, you can skip this section.

2. generate ssh key-pair
```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```
insert your email address there and 

## Configuring AWS
1. login
2. create user
3. give ec2 full access
4. copy secret key
5. upload ssh keys

```
aws configure
```

## Install boto3
```
sudo pip install boto3
```

## Download and configure playbook
```
cat ~/.ssh/id_rsa.pub > files/admins.pub
cp files/admins.pub files/ssh_validator.pub
```

## Deployment

### Create instance

### Configure instance

### Close external access
