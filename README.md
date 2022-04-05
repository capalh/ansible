# What's Ansible

A powerful automated deployment tool

### Prepare Ansible test environment

1. Deploy three Linux host by levarege Vagrant. Refer setup.sh script.
2. Install ansible and its dependency. Refer setup.sh script. 
2. Use VSCode to sync the folder. Install the sftp plugin under VScode. 

Sample sftp.json file

```
{
    "name": "My Server",
    "host": "192.168.200.11",
    "protocol": "sftp",
    "port": 22,
    "username": "vagrant",
    "privateKeyPath": "/Users/labadmin/DevOps/ansible/.vagrant/machines/master/virtualbox/private_key",     # Use private key to auth
    "remotePath": "/home/vagrant/ansible",                                                                  # Set the correct path, need to mkdir the ansible folder first
    "uploadOnSave": true
}

```


### Inventory 

The `Inventory` is used to define the list of the hosts

1. A sample inventory file - Try to ping the host-2 using ansible

```
host-2  ansible_connection=ssh  ansible_user=vagrant    ansible_ssh_pass=vagrant
```
Then run `ansible all -m ping -i inventory.ini`, got the response:

```
host-2 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

ping - the ping here is Ansible ping rather than TCP/IP ping
all - excute all hosts 

2. We can also specify a host for ping test `ansible host-3 -m ping -i inventory.ini`

```
$ ansible host-3 -m ping -i inventory.ini
host-3 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

3. Group of hosts, then `ansible web1 -m ping -i inventory.ini`, 

```
[web1]
host-2  ansible_connection=ssh  ansible_user=vagrant    ansible_ssh_pass=vagrant

[web2]
host-3  ansible_connection=ssh  ans  ible_user=vagrant    ansible_ssh_pass=vagrant
```
or using Regular expression

```
[web1]
host-[2:3]  ansible_connection=ssh  ansible_user=vagrant    ansible_ssh_pass=vagrant
```

4. Use SSH-key to auth. In real world, we don't want to input password in the inventory file

Step1: Use `ssh-keygen` to genreate key-pair
Step2: Use `ssh-copy-id -i ansible host-2` to copy the public key to the host-2 and host-3
Step3: Login use `ssh -i ~/.ssh/ansible host-2` without input password

```
ansible web1 -m ping -i inventory.ini --private-key=~/.ssh/ansible
```



### Playbook

Playbooks are Ansible's configuration, deployment, and orchestration language, and use YAML data structure. 

1. Use playbook. Sample file 

```
- hosts: web1
  name: play-test
  tasks:
    - name: check host connection
      ping: 
```

2. Excute the playbook `ansible-playbook playbook.yaml -i inventory.ini --private-key=~/.ssh/ansible`
3. Result ->

```
$ ansible-playbook playbook.yaml -i inventory.ini --private-key=~/.ssh/ansible

PLAY [play-test] *****************************************************************************************************************************************************************************

TASK [Gathering Facts] ***********************************************************************************************************************************************************************
ok: [host-3]
ok: [host-2]

TASK [check host connection] *****************************************************************************************************************************************************************
ok: [host-2]
ok: [host-3]

PLAY RECAP ***********************************************************************************************************************************************************************************
host-2                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
host-3                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


### Ansible core concept 

1. Inventory - Define the host list
2. Playbook - Integrate various of mudules to excute kind of request
3. Module - Used to act kinds of request, there are lots of mudules. See the KB:

https://docs.ansible.com/ansible/2.9/modules/list_of_all_modules.html


### Ansible configuraton file - ansible.cfg

