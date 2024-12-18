
Cisco switches are commonly used in network infrastructures, and configuring them manually can be time-consuming and error-prone, especially when scaling networks. Ansible helps automate these tasks, reducing manual intervention and ensuring consistent


###  Installing Required Packages



### Configure Inventory

```
shahin@shahin:~/ansible-cisco$ echo "switch" >> inventories/hosts.ini
shahin@shahin:~/ansible-cisco$ echo "switch1 ansible_host=192.168.1.10 ansible_user=admin ansible_password=your_password ansible_network_os=ios" >> inventories/hosts.ini
shahin@shahin:~/ansible-cisco$ echo "switch2 ansible_host=192.168.1.11 ansible_user=admin ansible_password=your_password ansible_network_os=ios" >> inventories/hosts.ini
```

###  Define Global Variables
Create the ``group_vars/all.yml`` file to define global variables for your playbooks:
```
ansible_connection: network_cli
ansible_become: yes
ansible_become_method: enable
ansible_become_password: your_enable_password
```


### Create a Configuration Template
Create the configuration file you want to deploy to the target systems. For example, we are updating the cisco_switch_conf.yml file for the cisco 2960.

### Create a Playbook

Create the ```playbooks/cisco_switch_deploye.yml``` playbook to apply configurations:

```
---
- name: Deploy configuration to Cisco switches
  hosts: all
  gather_facts: no

  tasks:
    - name: Push configuration to switches
      ios_config:
        src: templates/cisco_switch_conf.yml

```
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

```
### Run the Playbook

```ansible-playbook -i inventories/hosts.ini templates/cisco_switch_conf.yml --check```


