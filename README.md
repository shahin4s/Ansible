ANSIBLE CONFIGURATION AND DEPLOYMENT 
---

```markdown
# Automating Cisco Switch Configuration with Ansible

## Introduction

Ansible is a powerful automation tool that simplifies the process of managing and configuring network devices such as Cisco switches. With Ansible, you can automate routine tasks like deploying configurations, managing users, and configuring SSH. This blog post will walk you through the process of setting up Ansible, creating a project structure, and automating the configuration of a Cisco switch.

## Prerequisites

Before we begin, ensure you have the following requirements:

- Ansible (core 2.12.2) installed on your local machine.
- Python 3.8 installed on your local machine.
- SSH enabled on your Cisco switch with password-based login.

### Installing Ansible

You can install Ansible on a Linux machine using the following commands:

```bash
# Update package list and install prerequisites
shahin@shahin:~$ sudo apt update
shahin@shahin:~$ sudo apt install software-properties-common

# Add Ansible PPA and install Ansible
shahin@shahin:~$ sudo add-apt-repository --yes --update ppa:ansible/ansible
shahin@shahin:~$ sudo apt install ansible
```

To verify that Ansible is installed correctly, run:

```bash
shahin@shahin:~$ ansible --version
```

You should see output similar to the following:

```
ansible 2.9.6
config file = /etc/ansible/ansible.cfg
configured module search path = ['/home/shahin/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
ansible python module location = /usr/lib/python3/dist-packages/ansible
executable location = /usr/bin/ansible
python version = 3.8.10 (default, Mar 15 2022, 12:22:08) [GCC 9.4.0]
```

## Project Setup

To organize your playbooks and configurations logically, create the following directory structure:

```
ansible-project/
├── inventories/
│   ├── production/
│   │   ├── group_vars/
│   │   │   ├── all.yml
│   │   │   ├── webservers.yml
│   │   ├── hosts
│   ├── staging/
│       ├── hosts
├── roles/
│   ├── common/
│   │   ├── tasks/
│   │   ├── handlers/
│   │   ├── templates/
│   │   ├── files/
│   │   ├── vars/
│   │   ├── defaults/
│   │   ├── meta/
├── playbooks/
│   ├── site.yml
│   ├── webservers.yml
├── ansible.cfg
└── README.md
```

- `inventories/`: Stores the inventory files for different environments (e.g., production, staging).
- `roles/`: Contains reusable roles to organize playbooks.
- `playbooks/`: Holds playbooks that define tasks for configuration or automation.
- `ansible.cfg`: The Ansible configuration file.

## Creating an Inventory File

An inventory file lists the hosts (such as servers or switches) you want to manage. It defines how to connect to them (via SSH, with the appropriate user and IP address).

Create an inventory file in the `inventories/production/` directory:

```bash
shahin@shahin:~/ansible-project$ touch inventories/production/hosts.ini
shahin@shahin:~/ansible-project$ echo "[mail]" >> inventories/production/hosts.ini
shahin@shahin:~/ansible-project$ echo "mail01 ansible_host=10.212.0.108 ansible_user=shahin" >> inventories/production/hosts.ini
shahin@shahin:~/ansible-project$ echo "mail02 ansible_host=10.212.0.109 ansible_user=shahin" >> inventories/production/hosts.ini
```

### Configuring the Ansible Config File

In the `ansible.cfg` file, configure the default inventory and disable host key checking for SSH:

```ini
[defaults]
inventory = inventories/production/hosts.ini
remote_user = 
host_key_checking = False
timeout = 30
ask_user = True
ask_pass = True
```

## Test Connectivity

Test that Ansible can connect to the hosts using the `ping` module:

```bash
shahin@shahin:~/ansible-project$ ansible mail01 -m ping
```

You should see:

```
mail01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

## Creating a Configuration File

Create the configuration file you want to deploy to the target systems. For example, we are updating the `resolv.conf` file for the mail servers.

```bash
shahin@shahin:~/ansible-project$ vim roles/files/resolv.conf
```

Add the following content to `resolv.conf`:

```txt
nameserver 10.212.0.53
search shahin.com
```

## Writing the Playbook

Now, write the Ansible playbook to deploy the configuration file to the target system. Create the `resolv_conf_deploy.yml` playbook:

```yaml
- name: Deploy resolv.conf file
  hosts: mail01
  become: yes
  tasks:
    - name: Backup resolve.conf
      copy:
        src: /etc/resolv.conf
        dest: /etc/resolv.conf.bkp
        remote_src: yes

    - name: Copy resolv.conf to the target system
      copy:
        src: /home/shahin/ansible-project/roles/files/resolv.conf
        dest: /etc/resolv.conf
        owner: root
        group: root
        mode: '0644'
```


## Running the Playbook

Run the playbook to deploy the configuration:

```bash
shahin@shahin:~/ansible-project$ ansible-playbook -i inventories/production/hosts.ini playbooks/resolv_conf_deploye.yml
```

### Expected Output:

```
PLAY [Deploy resolv.conf file] ********************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************
ok: [mail01]

TASK [Backup resolve.conf] ************************************************************************************************************
changed: [mail01]

TASK [Copy resolv.conf to the target system] ******************************************************************************************
ok: [mail01]

PLAY RECAP ****************************************************************************************************************************
mail01                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

This confirms that the `resolv.conf` file was successfully copied to the target system.


## Configuring SSH on Cisco Switch

To enable SSH and set login credentials on your Cisco switch, follow these steps:

1. Connect to the switch via console cable and terminal emulator (such as PuTTY).
2. Enter configuration mode and set up the SSH username and password:

```bash
switch# enable
switch# conf t
switch(config)# username shahin privilege 15 secret shahin123
switch(config)# crypto key generate rsa modulus 2048
switch(config)# line vty 0 15
switch(config)# transport input ssh
switch(config)# copy running-config startup-config
switch(config)# exit

