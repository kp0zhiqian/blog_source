---
title: "Ansible Reading Notes - Playbook&Role"
date: 2021-10-02T00:10:57+08:00
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

## Playbook: A yaml file that can be excuted by ansible
Before playbook, we need to understand YAML: [what is yaml](https://www.redhat.com/en/topics/automation/what-is-yaml)

### CMD
- ```ansible-playbook deploy.yml```
- ```ansible-playbook playbook.yml --verbose```
- ```ansible-playbook playbook.yml --list-hosts```
- ```ansible-playbook playbook.yml -f 10```

### Basic Playbook Structure
playbook is basically about 3 questions:
1. what hosts do we want to run playbook on?
2. what tasks do we want to run?
3. what aftercare tasks do we want to run?

#### Q1: what hosts do we want to run playbook on?
- host: IP, hostname or all
- user: remote uses to use
- become: if we want to change user, yes or no
- become_method: sudo, su, pbrun, pfexec, doas
- become_user: root or other usename

we can also use `--ask-become-pass` to promot password

#### Q2: what tasks do we want to run?
- task run from top to down, if any task has any failure, the whole playbook will stop.
- each task will call a module
- each task need a name to provide readability

#### Q3: what aftercare tasks do we want to run?
handler is event of playbook, the exexcutation order of handler is follow the order you define them.

Example:
```yaml
handlers:
- name: call in every action
  debug: msg="xxxx"
```




### Logic Control in Playbook

#### `when`

- we use `when` to to excute certain task when meet some requirment.

  e.g.
  ```yaml
  # shutdown the system if it's Debian
  tasks:
    - name: "shut debian"
      command: /sbin/shutdown -t now
      when: ansible_os_family == "Debian"
  ```
- we can also use expressions in `when`
  e.g.
  ```yaml
  vars:
    epic: true
  
  tasks:
    - shell: echo "something"
      when: epci
      # when: not epic
      # when: foo is defined
      # when: foo is not defined

  ```

#### Loop

- we could use `with_items` to iterate list.

  e.g.
  ```yaml
  - name: add several users
    user: name={{ item }} state=present groups=wheel
    with_items:
      - testuser1
      - testuser2
  # or we can define a list
  vars:
    list1: ["user1", "user2"]
  tasks:
    - name: add several users
      user: name={{ item }} state=present groups=wheel
      with_items: "{{ list1 }}"
  ```

#### Block

we can use `block` to excute a series of tasks in one block of task
```yaml
tasks:
  - block:
      - yum: name={{ item }} state=installed
        with_items:
          - httpd
          - memcached
      - template: src=templates/src.j2 dest=/etc/foo.conf
      - service: name=bar state=started enabled=True
    when: ansible_distribution == 'CentOS'
    become: true
    become_user: root
```

block will make try-cache-finally easily
```yaml
tasks:
  - block:
      - yum: name={{ item }} state=installed
        with_items:
          - httpd
          - memcached
      - template: src=templates/src.j2 dest=/etc/foo.conf
      - service: name=bar state=started enabled=True
    when: ansible_distribution == 'CentOS'
    become: true
    become_user: root
    rescue: 
      - debug: msg='I got an error'
      - command: /bin/false
      - debug: msg='I also never execute'
    always:
      - debug: msg='this always excutes'
```

### Reuse of Playbook

#### `include`

- we could use `include` to use pre-written playbook yaml file.

  ```yaml
  # tasks/firewall_httpd_default.yml
  - name: insert firewalld rule for httpd
    firewalld: port=80/tcp permanent=true state=enabled immediate=yes

  # main.yml
  tasks:
    - include: tasks/firewall_httpd_default.yml
  ```
- we can also use params
  ```yaml
  # tasks/firewall_httpd_default.yml
  - name: insert firewalld rule for httpd
    firewalld: port={{ port }}/tcp permanent=true state=enabled immediate=yes

  # main.yml
  tasks:
    - include: tasks/firewall_httpd_default.yml port=432
  ```
- using `include` in the playbook's global space is not recommonded, sometimes it's unstable.

- `include` become more and more powerful and also more and more unstable.

### Tags

- basic tags

  we can use `tags` to tag a task,   and run the tagged task by `ansible-playbook example.yml --tags "packages"`
  ```yaml
  tags:
    - packages
  ```

- special tags
  - always
  - tagged
  - untagged
  - all

## Best Practice of Writing Playbook
follow below two principles
- use include and role to avoid duplicate code
- seperate big files into small files.


