---
title: "Using Unvault to Read Vaulted and Non Vaulted Files"
date: 2020-08-16T22:57:17-04:00
draft: false
author: serainville
tags:
  - ansible
  - vault
description: |
    Learn how to use the new Ansible Unvault plugin in a lookup to read the contents of any file, vaulted an non vaulted. 
---

Unvault is a plugin introcuded by the Ansible core team which allows you to read contents of any file, vaulted or not. Used within a `lookup` you can retrieve the contents of a specific file. Ansible we determine whether it is vualted or not and apply the appropriate action to read it.


## Create a regular file
The following file be used as an example non vaulted file that will have its contents read using `unvault` in a lookup. The file's name will be `foo.txt`, and it will have the following contents:

```text
hello, world!
```

## Lookup contents of file
In order to retrieve the contents of the file we use a lookup with the `unvault` plugin, and specify the path to the file on the Ansible controller's file system.

```yaml
- name: Read contents of foo.txt
  debug: msg="the value of foo.txt is {{lookup('unvault', '/etc/foo.txt')|to_string}}"
```

The output of the debug message will contain the contents of the `foo.txt` file.

```shell
hello, world!
```

## Create a vualted file
Our next example will be of a vaulted file named `bar.txt.vaulted`. Create a new vaulted file named `bar.txt.vaulted`.

```shell
ansible-vault create bar.txt.vaulted
```

The file will have the following contents.

```text
hello, world! I'm vaulted!
```

## Lookup contents of vaulted file
Using a lookup with the `unvault` plugin, we can read the contents of the vaulted file `bar.txt.vaulted`. In the example below, like in the non vualted file example above, we are outputting the contents of the file in a debug message.

```yaml
- name: Read contents of bar.txt.vaulted
  debug: msg="the value of bar.txt.vaulted is {{lookup('unvault', '/etc/bar.txt.vaulted')|to_string}}"
```

The debug message should print the following to screen.

```text
hello, world! I'm vaulted!
```
