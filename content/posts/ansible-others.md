---
title: "Ansible Reading Notes - Others"
date: 2021-10-04T00:10:57+08:00
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

## `lookup` to access external file or database
Basic usage:
```yaml
---
- hosts: all
  remote_user: root
  gather_facts: false
  vars:
    contents: "{{ lookup('file', 'data/plain.txt') }}"
```

we can also read other data:`lookup('file|env|pipe|template|ini|csvfile|dig|')`

## filter
filter excute commands in the management node and get responds.

Usage of filters:
```yaml
- hosts: localhost
  remote_user: root
  gather_facts: false
  vars:
    my_test_string: "This is the testing string"
  tasks:
    - name: "quote {{ my_test_string }}"
      debug: msg="echo {{ my_test_string | quote }}"
```

different filters:
- quote
- default(n)
- default(omit)
- mandatory
- bool
- ternary
- basename
- dirname
- expanduser
- realpath
- relpath
- splitext
- b64encode
- to_uuid
- hash('sha1'|'md5'|)
- checksum
- comment()
- ipaddr
- ipv4
- ipv6
- to_json
- to_yaml

## Assert
Usage is just like filters
- match
- search
- version_compare
- issubset
- issuperset
- is_dir
- is_file
- is_link
- exist

assert the cmd result
- failed
- successed
- changed
- skipped

## Plugin
