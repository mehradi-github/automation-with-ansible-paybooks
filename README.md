# Automation with Ansible


<!-- TABLE OF CONTENTS -->
## Table of Contents
- [Automation with Ansible](#automation-with-ansible)
  - [Table of Contents](#table-of-contents)
  - [What is Ansible?](#what-is-ansible)
  - [Installing Ansible](#installing-ansible)
    - [Installing Ansible from PyPI using pip](#installing-ansible-from-pypi-using-pip)
    - [Installing Ansible on specific operating systems](#installing-ansible-on-specific-operating-systems)
      - [Installing Ansible on Ubuntu](#installing-ansible-on-ubuntu)
      - [Installing Ansible on Amazon Linux 2](#installing-ansible-on-amazon-linux-2)
  - [Ansible playbooks](#ansible-playbooks)
    - [Handlers: running operations on change](#handlers-running-operations-on-change)
    - [Notifying handlers](#notifying-handlers)
    - [Loops](#loops)
    - [Import a task list](#import-a-task-list)
    - [Delegating tasks](#delegating-tasks)
    - [Template](#template)
    - [Vault](#vault)
    - [Roles](#roles)

## What is Ansible?
[Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) automates the management of remote systems and controls their desired state.A basic Ansible environment has three main components:
- **Control node:**
A system on which Ansible is installed. You run Ansible commands such as ansible or ansible-inventory on a control node.

- **Managed node:**
A remote system, or host, that Ansible controls.

- **Inventory:**
A list of managed nodes that are logically organized. You create an inventory on the control node to describe host deployments to Ansible. 

<img src="public/assets/images/ansible_basic.svg" alt="Basic components of an Ansible environment include a control node, an inventory of managed nodes, and a module copied to each managed node." width="266" height="321">


## Installing Ansible 
### Installing Ansible from PyPI using pip
The ansible package can always be installed from PyPI [using pip](https://github.com/mehradi-github/ref-python#installing-packages-using-pip) on most systems but it is also packaged and maintained by the community for a variety of Linux distributions. [more detailes](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html#intro-installation-guide).

### Installing Ansible on specific operating systems
Installing Ansible on specific operating systems: [Fedora, CentOS, Debian, Ubuntu, Windows](https://docs.ansible.com/ansible/latest/installation_guide/installation_distros.html)

#### Installing Ansible on Ubuntu
```sh
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

#### Installing Ansible on Amazon Linux 2
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

## Ansible playbooks
Some of important commands:

```sh
which ansible-playbook
# Generating ansible.cfg
ansible-config init --disabled > ansible.cfg
ansible-config init --disabled -t all > ansible.cfg

sudo chmod +x sample-playbook.yml
#!~/.local/bin/ansible-playbook
./sample-playbook.yml 

# --check Dry Run
ansible-playbook sample-playbook.yml -e "y=2" --check
# --extra-var via json

echo "{'y':36}" >> vars.json
ansible-playbook sample-playbook.yml -e "@vars.json" --check

# -k, --ask-pass: ask for connection password
# -K, --ask-become-pass: ask for privilege escalation password
ansible-playbook sample-playbook.yml -e "{'y':2,'firstname':'alex'}" --check

#tags
ansible-playbook tag-playbook.yml -t common
ansible-playbook tag-playbook.yml --tags first,second
ansible-playbook tag-playbook.yml --tags --list-tags

```
More Details: [List Filters](https://docs.ansible.com/ansible/2.8/user_guide/playbooks_filters.html#list-filters)
### Handlers: running operations on change
Sometimes you want a task to run only when a change is made on a machine. For example, you may want to restart a service if a task updates the configuration of that service, but not if the configuration is unchanged. Ansible uses **[handlers](https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html)** to address this use case. Handlers are tasks that only run when notified.

### Notifying handlers
Tasks can instruct one or more handlers to execute using the notify keyword. The notify keyword can be applied to a task and accepts a list of handler names that are notified on a task change. Alternately, a string containing a single handler name can be supplied as well. The following example demonstrates how multiple handlers can be notified by a single task:
### Loops
Ansible offers the **[loop](https://docs.ansible.com/ansible/latest/user_guide/playbooks_loops.html)**, with_<lookup>, and until keywords to execute a task multiple times.

### Import a task list
  - [ansible.builtin.import_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/import_tasks_module.html) module – Import a task list
  - [ansible.builtin.include_tasks](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/include_tasks_module.html) module – Dynamically include a task list

### Delegating tasks
If you want to perform a task on one host with reference to other hosts, use the [delegate_to](https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html) keyword on a task.
There is also a shorthand syntax that you can use on a per-task basis: **local_action**. Using the shorthand syntax for delegating to 127.0.0.1

**run_once:** boolean that will bypass the host loop, forcing the task to attempt to execute on the first host available and afterwards apply any results and facts to all active hosts in the same batch.

### Template
[ansible.builtin.template](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/template_module.html) module – [Template](https://jinja.palletsprojects.com/en/3.1.x/) a file out to a target host


### Vault
Ansible Vault encrypts variables and files so you can protect sensitive content such as passwords or keys rather than leaving it visible as plaintext in playbooks or roles. To use Ansible Vault you need one or more passwords to encrypt and decrypt content. If you store your vault passwords in a third-party tool such as a secret manager, you need a script to access them. Use the passwords with the [ansible-vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) command-line tool to create and view encrypted variables, create encrypted files, encrypt existing files, or edit, re-key, or decrypt files. You can then place encrypted content under source control and share it more safely.

```sh
ansible-vault -h
ansible-vault create keys.yml
ansible-vault view keys.yml
ansible-vault edit keys.yml
ansible-vault rekey keys.yml

ansible-playbook vault-playbook.yml  --ask-vault-pass
ansible-playbook vault-playbook.yml  --vault-password-file mypass
ansible-playbook vault-playbook.yml  --vault-id mypass
# ANSIBLE_VAULT_PASSWORD_FILE=~/.vault_pass.txt

```
### Roles
[Roles](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html) let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content in roles, you can easily reuse them and share them with other users.

Using the [ansible-galaxy](https://galaxy.ansible.com/docs/contributing/creating_role.html#creating-roles) command line tool that comes bundled with Ansible, you can create a role with the init command. 

```sh
which ansible-galaxy
ansible-galaxy init test-role-1 --offline
```


