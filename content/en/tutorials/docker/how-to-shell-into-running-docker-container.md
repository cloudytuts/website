---
title: "How to Shell Into Running Docker Container"
date: 2020-08-21T09:09:15-04:00
draft: false
author: serainville
tags:
  - docker
description: |
    Learn how to load an interactive shell inside of a running Docker container based on Alpine, Debian, or Ubuntu in order to perform operational tasks.
---

Understanding how to interact with a running Docker container is a foundamental skill you will need in a containerized environment. You will need to understand how to shell into a running container to troubleshoot problems or verify configurations, for example.

## Whats Covered
* Learn how to interactively shell into a container
* Understand emphemeral container states

## Interactive Shell
An interactive shell is what we use to execute commands on a Linux host, with Bash being one of the most popular. Nearly all Docker containers are configured to allow running Bash or similar shell.

To run an interactive session with a running Docker container we use the `docker exec` command with the `-i` and `-t` flags, or `-it` for shorter. The `-i` flag allow us to interact with the container, while the `-t` flag is used to open a terminal into the container.

* The `-i` flag, or `--interactive`, instructs Docker to keep STDIN open allowing you to continuously interact with the container.
* The `-t` flag, or `--tty`, allocates a pseudo-TTY which creates the terminal shell.

The following syntax show you how to shell into a running container. 

```shell
docker exec -it <container-id|container-name> <path/to/shell>
``` 

Depending on the base image used to run your container the shell path may differ.

### Debian and Ubuntu Containers
Debian, Red Hat, and Ubuntu all use the common Bash shell. To open an interactive bash shell into a container based off of any of these Linux distributions, we would set the shell path as `/bin/bash`/

For example, to open an interactive Bash shell for an Debian, Red Hat, or Ubuntu based container with the ID `abc123` you would run the following command:

```shell
docker exec -it abc123 /bin/bash
```

### Alpine-based Containers
Alpine Linux uses a different shell than Debian, Read Hat, and Ubuntu. It uses the Alpine Shell, or `ash` for short. 

To open an interactive shell with an Alpine Linux based container, we would execute the following command.

```shell
docker exec -it abc123 /bin/ash
```

### Exiting an Interactive Shell
To exit from an interactive shell you can typically just type `exit` and press enter.

```shell
exit
```

What ever changes to the state of your running container that were made while in the interactive shell will remain.


## Ephemeral
Running Docker containers are ephemeral, meaning their state is wiped as soon as the container stops. Any changes executed against a running container will be lost as soon as the container stops. This means you should reserve interactive shells for troubleshooting and information gathering only.

However, if you are performing actions against files on a mounted volume in your container, those changes will persist beyond the life of your running container.

To permenantly apply your state changes you must update the image your container is based on.
