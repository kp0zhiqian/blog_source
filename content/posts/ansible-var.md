---
title: "Ansible - Var"
date: 2021-10-01T00:10:57+08:00
draft: false
category: notes
tags:
    - ansible
keywords:
    - ansible
    - automation
    - system
---

# Var
## Define Var in Playbook
```bash
vars:
  http_port: 80
...
tasks:
- name: xxx
  firewalld: port={{http_port}}
  
```

## Define Var in File
file:
```bash
http_port:80
```

usage
```bash
- hosts: web
  vars_files:
    - vars/server_vars.yml
```

## Var Trap
sometimes YAML and ansible playbook cannot work together. when it report syntex error, you could try add quote. 

## Facts
Facts are the information of remote hosts collected by 'setup' module.

we can directly use the vars in `ansible all -m setup -u root` like `{{some_vars}}`

the vars we collected will be in a json format, we could use `.` to access different level of json like `{{ansible_ens3.ipv4.address}}`

## Register Vars

we can register var in the playbook and use it later.

```bash
- hosts: web
  tasks:
    - shell: ls
      register: result
      ignore_errors: True
    - shell: echo "{{ result.stdout }}"
      when: result.rc == 5
```

## Pass Vars Through CMD
we could use
```bash
ansible-playbook xxx.yml --extra-vars "hsots=web user=root"
```
to pass vars to playbook.yml

we could use vars in playbook just like other type of vars

## Var Scope
- global
    - vars in the ansible.cfg
    - env vars
    - from cli
- play
    - `var` keywords in playbook
    - `include_vars` keyword
    - `default/main.yml` or `vars/main.yml` in role
- host
    - vars defined in the host inventory list
    - remote host system var
    - registered vars

## Var Precedence
[ekreutz/ansible_variable_precedence.md](https://gist.github.com/ekreutz/301c3d38a50abbaad38e638d8361a89e)

from the least to the most important

- role defaults
- inventory file or script group vars
- inventory group_vars/all
- playbook group_vars/all
- inventory group_vars/*
- playbook group_vars/*
- inventory file or script host vars
- inventory host_vars/*
- playbook host_vars/*
- host facts
- play vars
- play vars_prompt
- play vars_files
- role vars (defined in role/vars/main.yml)
- block vars (only for tasks in block)
- task vars (only for the task)
- role (and include_role) params
- include params
- include_vars
- set_facts / registered vars
- extra vars (always win precedence)