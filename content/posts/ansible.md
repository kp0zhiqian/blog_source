---
title: "Ansible Reading Notes"
date: 2020-09-30T00:10:57+08:00
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

## Host Inventory
- default host file: `/etc/ansible/hosts`
- We can use [group_name] to setup group, but it's not required
    ```bash
    [webservers]
    foo.example.com
    bar.example.com
    ```

## CLI tools
- `ansible <host> [options]`
- `ansibel <host> -m <modules>`

## Playbook: A yaml file that can be excuted by ansible 
- `ansible-playbook deploy.yml`
- keywords:
    - `hosts`: host IP or group name or all
    - `remote_user`: excute using some uses
    - `vars`: variables
    - `tasks`: core of playbook, define action, every action can call a modules

## Module
Ansible Module: `module` in ansible is `cmd` in bash
- Use module in cli
    - `-m <module_name>`
    - `-a <module_param>`
- Use module in playbook
    - `module_name: module_option=option_value`
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
