---
title: "Ansible Reading Notes - Basic"
date: 2021-09-30T00:10:57+08:00
draft: false
category: notes
tags:
    - ansible
    - system
    - automation
keywords:
    - ansible
    - automation
    - system
---

## Installation&setup
- Redhat/Centos need to Install EPEL, fedora has it in its default source
- Generate ssh, copy ssh to remote, save remote hosts to known_hosts

## Configuration
default configuration file: ```/etc/ansible/ansible.cfg```

basic options: 
- ```inventory```: path to host list file
- ```library```: path to extra module
- ```remote_tmp```: remote host temp files path
- ```local_tmp```: management node temp fiels path

### configuration file priority
Ansible will try to find configuration file in below order:
1. ANSIBLE_CONFIG (env var)
2. ansible.cfg (current directory)
3. .ansible.cfg (home directory)
4. /etc/ansible/ansible.cfg


### Host Inventory
- default host file: `/etc/ansible/hosts`
    - use ```-i``` or ```--inventory-file``` to load separate host list
    - ```ansible-playbook -i hosts site.yml```
- We can use [group_name] to setup group, but it's not required
    ```bash
    [webservers]
    foo.example.com
    bar.example.com
    ```
    - we can also nested the groups
    ```bash
    [atlanta]
    host1
    host2
    [raleigh]
    host2
    host3

    [southeast:children]
    atlanta
    raleigh
    [usa:children]
    southeast
    ```
- we can also assign connection param in inventory file:
    ```bash
    [targets]
    localhost           ansible_connection=local
    other1.example.com  ansible_connection=ssh  ansible_user=root   ansible_user=user1
    ```
- create vars for a group
    ```bash
    [atlanta]
    host1
    host2

    [atlanta:vars]
    ntp_server=ntp.atlanta.example.com
    proxy=proxy.atalanta.example.com

## CLI tools
- `ansible <host> [options]`
- `ansibel <host> -m <modules>`

## Module
Ansible Module: `module` in ansible is `cmd` in bash
- Use module in cli
    - `-m <module_name>`
    - `-a <module_param>`
- Use module in playbook
    - `module_name: module_option=option_value`
- type of module
  - Core module: doesn't need to download and install, core module will be well tested
  - extra module: need to download and install, maybe bug
    1. download: git clone xxxxx
    2. change configuration file `/etc/ansible/ansible.cfg`, add `library = /home/$pathToExtraModule`, or change ansible.cfg in current directory or change ANSIBLE_LIBRARY
  - check modules `ansible-doc module_name`
- common modules
    - ping
        - not only check connectivity, also check ssh availiability and python version
        - don't need any param
    - debug
        - similer to `echo`, it can print some debugging msg
        - `msg: <some info>`
        - `var: <some variables>`
    - copy
	    - copy files from the mgt host to remote host
	    - `mode` to set permission
	    - `backup` can backup files before copy
		- `validate` can set some script to validate
	- template
		- copy files, but there're some dynamic variables to change in the files
		- use `{{ var_name }}` in files to access remote host env variables and vars in playbook's var section.
		- also support permission and user/group/validate...
	- file
		-  configure files, symlinks and folders' permission, create and delete them.
		-  `mode` to configure permission
		-  `src` and `dest` to create symlinks
		-  `state: touch` to create file
		-  `state: directory` to create folder
	- user
		- create user and attribute
		- `groups` to add user to group
		- `state: absent` to delete user
		- change user attribute like ssh key, and expires
    - yum
	    - use `state:latest` to install the latest package
	    - use `state:absent` to delete the package
	    - use `state:present` and `name:<pkgname_with_version>` to install a certain version
	    - use `enablerepo: <some_repo>` to install package from a certain repo
	    - use `name: @Development` tools to install a group of packages
	    - use `name: <path_to_rpm_file>` to install a local package
	    - use `name: <url_to_rpm>` to install a package from URL
	- service
        - `state: started` to start servvce
        - `state: stopped` to stop service
        - `state: restarted` to restart service
        - `state: reloaded` to reload service
        - `enable: yes` to start it when the host online
        - `args: eth0` in `network` service to enable network's interface
    - firewalld
        - add rules for service
        ```yaml
        - firewalld:
            service: https
            permanent: true
            state: enabled
        ```
        - add rules for port
        ```yaml
        port: 8081/tcp
        ```
        - other rules `rich_rule, source, zone, interface, masquerade`
    - shell: issue cmds in remote host
        - support operators like `HOME, < > | ; & && >> ||`
        - call script `- shell: script.sh >> log.txt`
        - use `args:` to 
            - `chdir` change working directory
            - `creates` excute only when file non-exist
            - `executable` use certain bash
    - command: similar to shell, but doesn't support operator


