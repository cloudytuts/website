---
title: "How to Install Ansible on Ubuntu Debian"
date: 2020-09-01T17:22:51-04:00
draft: false
author: serainville
tags:
    - ansible
    - pip
    - pip3
    - ubuntu
    - debian
description: |
    Learn how to install Ansible on Ubuntu and Debian with Python Pip3
---
In this tutorial, you will learn how to install Ansible on Ubuntu and Debian. We will cover using the default distrubtion repositories, as well as Python Pip3, as well as expanding on different ideas on using these methods in Agile DevOps environments.


## Install Ansible 
The [Ansible Docs](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html) provide a good resource for understanding the basics of installing Ansible. 

### Ubuntu and Debian Apt Repository
Ansible is available from the official Ubuntu repository and is the easiest method of installing it. 

To check which version of Ansible is available for your version of Ubuntu you can use the `apt search` command. For example, the following search was performed on Ubuntu 20.04 for Ansible.
```shell
sudo apt search ansible
```
```shell {hl_lines=["3-4"]}
Sorting... Done
Full Text Search... Done
ansible/focal 2.9.6+dfsg-1 all
  Configuration management, deployment, and task execution system

ansible-doc/focal 2.9.6+dfsg-1 all
  Ansible documentation and examples

ansible-lint/focal 4.2.0-1 all
  lint tool for Ansible playbooks

```

From the results we can see that Ansibl 2.9.6 is available, which is already several releases behind upstream. At the time of writing this tutorial, Ansible version 2.9.X is already at 2.9.13, and version 2.10 is also out. 

The problem you will always face with installing Ansible from your distrubtion's package repository, especially for distros that pride themselfves on stability, is your packages will always be out of date. This statement is true for even bug releases.

{{< note >}}
If you are running multiple distro releases in your environment, each install will use a different version of Ansible when installed from the distro's package repository. This is not ideal where you want predictable and reliable results from your playbooks.
{{< /note >}}

To install Ansible from the Ubuntu or Debian repositories, simply run an `apt install` command.

```shell
sudo apt install ansible
```




### Python Pip
A better solution over installing Ansible from Ubuntu's or Debian's package repository is to install it using Pip or Pip3. As Ansible is a Python project, each release is published to Pypi and that means you have much more flexibility over which versions you want to install.

Install Pip3 onto your Debian or Ubuntu server.
```shell
sudo apt install python-pip3
```

Install Ansible using Pip3
```shell
pip3 install --user ansible
```

In comparison to the Ubuntu 20.04 package repository, Pip3 defaults to a much more recent release. At the time of this writing Ansible 2.9.13 was the most recently available release, and as such is being installed.

```shell
Collecting ansible
  Downloading ansible-2.9.13.tar.gz (14.3 MB)
     |████████████████████████████████| 14.3 MB 2.0 MB/s 
Requirement already satisfied: PyYAML in /usr/lib/python3/dist-packages (from ansible) (5.3.1)
Requirement already satisfied: cryptography in /usr/lib/python3/dist-packages (from ansible) (2.8)
Requirement already satisfied: jinja2 in /usr/lib/python3/dist-packages (from ansible) (2.10.1)
Building wheels for collected packages: ansible
  Building wheel for ansible (setup.py) ... done
  Created wheel for ansible: filename=ansible-2.9.13-py3-none-any.whl size=16181748 sha256=64ea86ef7a346b9121b01a158d6af86e0f73872eec28d6a1832e95f3c3dbc45f
  Stored in directory: /home/srainville/.cache/pip/wheels/08/79/b6/1b4fabbf813ee59f86d46389bf70ba95017c91af6daee1acf4
Successfully built ansible
Installing collected packages: ansible
Successfully installed ansible-2.9.13
```

If you would rather install a specific version of Ansible, rather than the latest, you can specify a version. 
```shell
pip3 install --user ansible==2.9.12
```

To view a full list of available versions visit (Ansible's Pypi history page)[https://pypi.org/project/ansible/#history]. Alternatively, a quick and dirty way to force Pip3 to output available versions is to specify an invalid version.

```shell
pip3 install ansible==
```
```shell
ERROR: Could not find a version that satisfies the requirement ansible== (from versions: 1.0, 1.1, 1.2, 1.2.1, 1.2.2, 1.2.3, 1.3.0, 1.3.1, 1.3.2, 1.3.3, 1.3.4, 1.4, 1.4.1, 1.4.2, 1.4.3, 1.4.4, 1.4.5, 1.5, 1.5.1, 1.5.2, 1.5.3, 1.5.4, 1.5.5, 1.6, 1.6.1, 1.6.2, 1.6.3, 1.6.4, 1.6.5, 1.6.6, 1.6.7, 1.6.8, 1.6.9, 1.6.10, 1.7, 1.7.1, 1.7.2, 1.8, 1.8.1, 1.8.2, 1.8.3, 1.8.4, 1.9.0.1, 1.9.1, 1.9.2, 1.9.3, 1.9.4, 1.9.5, 1.9.6, 2.0.0.0, 2.0.0.1, 2.0.0.2, 2.0.1.0, 2.0.2.0, 2.1.0.0, 2.1.1.0, 2.1.2.0, 2.1.3.0, 2.1.4.0, 2.1.5.0, 2.1.6.0, 2.2.0.0, 2.2.1.0, 2.2.2.0, 2.2.3.0, 2.3.0.0, 2.3.1.0, 2.3.2.0, 2.3.3.0, 2.4.0.0, 2.4.1.0, 2.4.2.0, 2.4.3.0, 2.4.4.0, 2.4.5.0, 2.4.6.0, 2.5.0a1, 2.5.0b1, 2.5.0b2, 2.5.0rc1, 2.5.0rc2, 2.5.0rc3, 2.5.0, 2.5.1, 2.5.2, 2.5.3, 2.5.4, 2.5.5, 2.5.6, 2.5.7, 2.5.8, 2.5.9, 2.5.10, 2.5.11, 2.5.12, 2.5.13, 2.5.14, 2.5.15, 2.6.0a1, 2.6.0a2, 2.6.0rc1, 2.6.0rc2, 2.6.0rc3, 2.6.0rc4, 2.6.0rc5, 2.6.0, 2.6.1, 2.6.2, 2.6.3, 2.6.4, 2.6.5, 2.6.6, 2.6.7, 2.6.8, 2.6.9, 2.6.10, 2.6.11, 2.6.12, 2.6.13, 2.6.14, 2.6.15, 2.6.16, 2.6.17, 2.6.18, 2.6.19, 2.6.20, 2.7.0.dev0, 2.7.0a1, 2.7.0b1, 2.7.0rc1, 2.7.0rc2, 2.7.0rc3, 2.7.0rc4, 2.7.0, 2.7.1, 2.7.2, 2.7.3, 2.7.4, 2.7.5, 2.7.6, 2.7.7, 2.7.8, 2.7.9, 2.7.10, 2.7.11, 2.7.12, 2.7.13, 2.7.14, 2.7.15, 2.7.16, 2.7.17, 2.7.18, 2.8.0a1, 2.8.0b1, 2.8.0rc1, 2.8.0rc2, 2.8.0rc3, 2.8.0, 2.8.1, 2.8.2, 2.8.3, 2.8.4, 2.8.5, 2.8.6, 2.8.7, 2.8.8, 2.8.9, 2.8.10, 2.8.11, 2.8.12, 2.8.13, 2.8.14, 2.8.15, 2.9.0b1, 2.9.0rc1, 2.9.0rc2, 2.9.0rc3, 2.9.0rc4, 2.9.0rc5, 2.9.0, 2.9.1, 2.9.2, 2.9.3, 2.9.4, 2.9.5, 2.9.6, 2.9.7, 2.9.8, 2.9.9, 2.9.10, 2.9.11, 2.9.12, 2.9.13, 2.10.0a1, 2.10.0a2, 2.10.0a3, 2.10.0a4, 2.10.0a5, 2.10.0a6, 2.10.0a7, 2.10.0a8, 2.10.0a9)
ERROR: No matching distribution found for ansible==
```

When an invalid version is specified, Pip3 will output an error along with all of the available releases. Once you have found the release you are looking for, simply specify in the `pip3 install` command.

```shell
pip3 install --user ansible==2.6.4
```

### Install from Source
Pypi is the best source for installing from a long history of releases, but you may want to pull from the upstream itself. Ansible is an open source project hosted on Github, which means you can download the source files directly.

To install directly from the official Github repository you will need Pip. For example, to install the latest development version you can specify it as part of the url with Pip3.

```shell
pip3 install --user git+https://github.com/ansible/ansible.git@devel
```


## Virtualenv
Virtualenv is used to isolate Python environments on a host system, allowing you to run and install multiple versions of an app or its dependencies. Ansible, being a Python project, can also benefit from being installed into a `virtualenv`. 

{{< note >}}
Installing Ansible in a `virtualenv` is a great way to isolate different Ansible installs. You may want, for example, to test a recent release or a pre-release without impacting your current installation.
{{< /note >}}

Create the virtualenv, if one does not exist already.
```shell
python -m virtualenv ansible
```

Activate the virtual environment
```shell
source ansible/bin/activate
```

Install Ansible into the virtual environment
```shell
pip3 install ansible==2.9.13
```

### Requirements File
When you are working in a within a highly collaborative environment that uses Ansible, playbooks and roles will be handled by different teams and shuffled around. Ensuring you are always using the same version of Ansible as everyone is crucial for ensuring predictive and reliable playbook runs. 

Python projects typically use `requirements.txt` files to lock versions of dependencies, when the code is shared between people and teams. This same approach can be used with Ansible.

If you are the creator an the Playbook, create a `requirements.txt` file in the root project directory. Add the following line to it, modifying the version to match the one you have installed locally.

```text
ansible==2.9.13
```

When someone downloads your playbook they can `pip install` using the `requirements.txt` file to ensure they are running the same version of Ansible as you.

```shell
pip3 --user install -r requirements.txt
```

### Requirements and Virtualenv
Even better is to mix virtualenv with requirements.txt files when sharing playbooks.

Create the virtualenv, if one does not exist already.
```shell
python -m virtualenv ansible
```

Activate the virtual environment
```shell
source ansible/bin/activate
```

Install Ansible into the virtual environment
```shell
pip3 install -r requirements.txt
```

Having to remember each of those steps might be difficult. A shell script can be used to setup the environment, and make everyone's life a little easier.

```shell
#!/bin/sh
python -m virtual ansible
source ansible/bin/activate
pip3 install -r requirements.txt
```

## Conclusion
In this tutorial, you have learned how to install Ansible on Ubuntu and Debian using different methods. While some of these are convered in the official Ansible documentation, other solutions exist beyond the docs. 

You've learned how to share playbooks in a highly collaborative environment by using pip3 and virtualenv. By doing so, everyone can reliably run each other's playbooks without problems. 

