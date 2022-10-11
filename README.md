# Automation with Ansible Playbooks


<!-- TABLE OF CONTENTS -->
## Table of Contents
- [Automation with Ansible Playbooks](#automation-with-ansible-playbooks)
  - [Table of Contents](#table-of-contents)
  - [What is Ansible?](#what-is-ansible)
  - [Installing Ansible on Amazon Linux 2](#installing-ansible-on-amazon-linux-2)
  - [Ansible Playbooks](#ansible-playbooks)

## What is Ansible?
[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) automates the management of remote systems and controls their desired state.A basic Ansible environment has three main components:
- **Control node:**
A system on which Ansible is installed. You run Ansible commands such as ansible or ansible-inventory on a control node.

- **Managed node:**
A remote system, or host, that Ansible controls.

- **Inventory:**
A list of managed nodes that are logically organized. You create an inventory on the control node to describe host deployments to Ansible. 

<img src="public/assets/images/ansible_basic.svg" alt="Basic components of an Ansible environment include a control node, an inventory of managed nodes, and a module copied to each managed node." width="266" height="321">

## Installing Ansible on Amazon Linux 2
```sh

python -V

#Install python and python-pip
sudo yum -y update
sudo yum install -y amazon-linux-extras
amazon-linux-extras | grep -i python
sudo amazon-linux-extras enable python3.8
sudo yum install python3.8
python3.8 --version



#Install from source
sudo yum -y update
sudo yum -y groupinstall "Development Tools"
sudo yum -y install openssl-devel bzip2-devel libffi-devel

sudo yum -y install wget
wget https://www.python.org/ftp/python/3.8.12/Python-3.8.12.tg

tar xvf Python-3.8.12.tgz
cd Python-3.8*/
./configure --enable-optimizations
sudo make altinstall



#Setting Default Python
sudo alternatives --install /usr/bin/python python /usr/bin/python3.8 1
sudo alternatives --install /usr/bin/python python /usr/bin/python2.7 2

#Confirm the new setting.
sudo alternatives --list | grep -i python
sudo alternatives --config python
#Ensuring pip is available
python -m pip -V

#If you see an error like No module named pip
curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
python get-pip.py --user


#Install ansible using pip check for version
python -m pip install  --user ansible
ansible --version
which ansible
python -m pip show ansible

#Create a user called ansadmin & grant sudo access to ansadmin user using "visudo" command (on Control node and Managed host)
useradd ansadmin
passwd ansadmin
echo "ansadmin ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

#Make sure that PasswordAuthentication yes in all servers under /etc/ssh/sshd_config file.
vi /etc/ssh/sshd_config 
sudo systemctl restart sshd

#Log in as a ansadmin user on master and generate ssh key (on Control node)
sudo su - ansadmin
ssh-keygen
#Copy keys onto all ansible managed hosts (on Control node)
ssh-copy-id ansadmin@<target-server>

#Validation test
# Create a directory /etc/ansible and create an inventory file called "hosts" add control node and managed hosts IP addresses to it.
vi /etc/ansible/hosts
ansible all -m ping
#Gathers facts about remote hosts
ansible all -m setup

```

More details: [Indexes of all modules and plugins](https://docs.ansible.com/ansible/latest/collections/all_plugins.html)

## Ansible Playbooks
Some of important commands:

```sh
which ansible-playbook
sudo chmod +x input-and-outputs-of-ansible.yaml
#!~/.local/bin/ansible-playbook
./input-and-outputs-of-ansible.yaml 

# --extra-var via json
# --check Dry Run
ansible-playbook input-and-outputs-of-ansible.yaml -e "{'y':2}" --check

```