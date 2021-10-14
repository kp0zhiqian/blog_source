---
title: "Ansible - Role"
date: 2021-10-03T00:10:57+08:00
draft: false
category: notes
tags:
    - ansible
keywords:
    - ansible
    - automation
    - system
---
## role

`role` is the ansible playbook's package.




we can also define `pre_tasks` and `post_tasks` to run tasks before and after role.

## Location of Role
- in the role directory of current directory
- in the `ANSIBLE_ROLES_PATH`
- in the `roles_path` of ansible configuration file.
- default directory: `/etc/ansible/roles`

## Use Role

```yaml
---
# file site.yml

- hosts: all
  remote_user: root
  roles:
    - simple
  tasks:
    - name: Taks in site.yml
      debug: msg="This is a task in site.yml"
```

```yaml
---
# foles/simple/tasks/main.yml

- name: Task in role
  debug: msg="Simple role"
```

### pre_task and post_task

we can use pre_task and post_task to do something before/after the role
### Use role's params

Use `vars` to pass params to role.

```yaml
---
- hosts: all
  remote_user: root
  roles:
    - role: my_role_with_vars
      vars:
        ex_param: "test1"
        in_param: "test2"
```

Use `when`
```yaml
---
- hosts: all
  remote_user: root
  roles:
    - role: my_role_with_vars
      vars:
        ex_param: "test1"
        in_param: "test2"
      when: "ansible_os_family == 'Debian'"
```

## How to write role

typical role directory structure:
```bash
# playbooks
site.yml
webservers.yml
fooservers.yml
roles/
    common/
        tasks/
        handlers/
        library/
        files/
        templates/
        vars/
        defaults/
        meta/
    webservers/
        tasks/
        defaults/
        meta/
```



> By default Ansible will look in each directory within a role for a main.yml file for relevant content (also `main.yaml` and `main`):
> - `tasks/main.yml` - the main list of tasks that the role executes.

> - `handlers/main.yml` - handlers, which may be used within or outside this role.

> - `library/my_module.py` - modules, which may be used within this role (see Embedding modules and plugins in roles for more information).

> - `defaults/main.yml` - default variables for the role (see Using Variables for more information). These variables have the lowest priority of any variables available, and can be easily overridden by any other variable, including inventory variables.

> - `vars/main.yml` - other variables for the role (see Using Variables for more information).

> - `files/main.yml` - files that the role deploys.

> - `templates/main.yml` - templates that the role deploys.

> - `meta/main.yml` - metadata for the role, including role dependencies.

There're 2 types of resources in role, `main.yml` under these directories and other `yaml` files.

## Dependencies
we can use
```yaml
dependencies:
- {role: common}
```
to add dependencies. 