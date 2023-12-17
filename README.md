# ANSIBLE Cisco Switch configurstion
Automating The Configuration Cisco Switch With Ansible

### Contents


#### Requirements
- Ansible [core 2.12.2] installed on node machine.
- Python3.8 intahll on node mathcin .
- SSH enabled on switch aslon passowrd base login.


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
  ansible [core 2.12.2]
  config file = /etc/ansible/ansible.cfg
  configured module search path = ['/home/ishraque/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python3/dist-packages/ansible
  ansible collection location = /home/ishraque/.ansible/collections:/usr/share/ansible/collections
  executable location = /usr/bin/ansible
  python version = 3.8.10 (default, Nov 26 2021, 20:14:08) [GCC 9.3.0]
  jinja version = 3.0.3
  libyaml = True
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
$ ansible-playbook -v playbook.yml --extra-vars "@password_secret.yml"
```


 

