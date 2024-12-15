# ANSIBLE Cisco Switch configuration
Automating The Configuration Cisco Switch With Ansible

### Contents


#### Requirements
- Ansible [core 2.12.2] installed on node machine.
- Python3.8 intahll on node mathcin .
- SSH enabled on switch also password base login.


#### Installing Ansible On Node Machine

Ansible install using the following commands:

```sh
shahin@shahin:~$ sudo apt update
shahin@shahin:~$ sudo apt install software-properties-common
shahin@shahin:~$ sudo add-apt-repository --yes --update ppa:ansible/ansible
shahin@shahin:~$ sudo apt install ansible
```
Check the ansible has been successfully installed
```sh
shahine@shahin:~$ ansible --version
  ansible 2.9.6
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/shahin/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Mar 15 2022, 12:22:08) [GCC 9.4.0]
```

### Set up a project directory with inventories, roles, and playbooks.
    Organize your files and playbooks logically for maintainability

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
  -  `inventories/`: Stores inventory files for different environments (e.g., production, staging).
  -   `roles/`: Contains reusable roles to organize playbooks.
  -   `playbooks/`: Playbooks defining tasks for configuration or automation.
  -    `ansible.cfg`: Ansible configuration file.
#### Create an Inventory File
An inventory file in Ansible is a file that lists the servers (hosts) you want to manage and defines how to connect to them. It can group hosts logically (e.g., by environment, role, or function) and assign variables that apply to specific hosts or groups.
An inventory file is essential because it tells Ansible:
                -  What hosts to manage.
                -  How to connect to those hosts (SSH user, keys, IPs, etc.).
  -  Formats for Inventory Files
          - INI Format (easiest and most common for small setups).
          - YAML Format (more structured, better for complex setups).



- Create a Basic Inventory File:
  
```sh
shahin@shahin:~/ansible-project$ touch inventories/production/hosts.ini
shahin@shahin:~/ansible-project$ echo "[mail]" >> inventories/production/hosts.ini
shahin@shahin:~/ansible-project$ echo "mail01 ansible_host=10.212.0.108 ansible_user=shahin" >> inventories/production/hosts.ini
shahin@shahin:~/ansible-project$ echo "mail01 ansible_host=10.212.0.109 ansible_user=shahin" >> inventories/production/hosts.ini

```

### Configure Ansible config file 

```sh
shahin@shahin:~/ansible-project$ cat ansible.cfg
[defaults]
inventory = inventories/production/hosts.ini
remote_user = 
#private_key_file =
#host_key_checking = False
host_key_checking = False
timeout = 30
ask_user = True
ask_pass = True

```


- Test Connectivity
Ensure that Ansible can connect to your hosts using the ping module:

```sh

shahin@shahin:~/ansible-project$ ansible mail01 -m ping
```
- Success
  
```sh
SSH password:
mail01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```
### create configuration file

Now We are updating resolv.conf to mail server. 
  creating config file 
```sh
shahin@shahin:~/ansible-project$ vim roles/files/resolv.conf
shahin@shahin:~/ansible-project$ echo "nameserver 10.212.0.53 " >> roles/files/resolv.conf
shahin@shahin:~/ansible-project$ echo "search shahin.com " >> roles/files/resolv.conf
```

#### write playbook 


```sh
shahin@shahin:~/ansible-project$ vim playbooks/resolv_conf_deploye.yml
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








####  SSH enabled on switch also enable passowrd base login.
 - Connect your console cable from your computer's serial port to the console port
 - Open a terminal emulator program on your computer, such as PuTTY (for Windows) 
 - Enabled ssh and set login credentials  
 
 ```sh
switch#enable
switch#conf t
switch(config)#username shahin privilege 15 secret shahin123
switch(config)#crypto key generate rsa modulus 2048
switch(config)#line vty 0 15
switch(config)#transport input ssh
switch(config)#copy running-config startup-config
switch(config)#exit

``

- Playbook can be run with the following command:

```sh
shahin@shahin:~/ansible-project$ ansible-playbook -i inventories/production/hosts.ini playbooks/resolv_conf_deploye.yml


- outpus
```SSH password:

PLAY [Deploy resolv.conf file] **********************************************************************************************************************************************************************************

TASK [Gathering Facts] ******************************************************************************************************************************************************************************************
ok: [mail01]

TASK [Backup resolve.conf] **************************************************************************************************************************************************************************************
changed: [mail01]

TASK [Copy resolv.conf to the target system] ********************************************************************************************************************************************************************
ok: [mail01]

PLAY RECAP ******************************************************************************************************************************************************************************************************
mail01                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```







#### How to Run The Playbook

- check the ssh connection to target device (NC -vz <IP address of host > 22)
- Add newtarget hosts to the ```inventory.yml``` file .
```sh 
---
all:
  children:
    user_hosts:
      hosts:
        192.168.6.25:
            ansible_user: shahin  
            username: shahin123

```

- Playbook can be run with the following command:
```sh 
shahin@shahin:~/ansible-project$ ansible-playbook -i inventories/production/hosts.ini playbooks/cisco_config_deploye.yml"
```


 

