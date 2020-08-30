---
title: "How to List, Start, and Stop Docker Containers"
date: 2020-08-29T23:24:19-04:00
draft: true
# author:
# tags:
description: |
    Learn how to list running containers, as well as how to start, stop, and delete containers using the docker command.
---


## Listing Running Containers
To view a list of running containers on your Docker host, you use the `docker ps` command. This will output a nicely formatted list of running containers, as well as their state.

```shell
docker ps
```
```shell
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
e51898f31223        3781fe35c6b8        "docker-entrypoint.s…"   4 days ago          Up 4 days                               k8s_postgres_postgres-778965fcc7-rc2wj_default_8c8e771b-5b1a-43ee-a9ea-e4af2c1e9319_0
bc97fe8bcd66        wordpress           "docker-entrypoint.s…"   5 days ago          Up 5 days                               k8s_wordpress_wordpress-69b59478c4-5lvkk_default_1c19bdb0-2dbc-4c74-9c23-fd1f19c785bc_0
17714162a329        718a6da099d8        "docker-entrypoint.s…"   5 days ago          Up 5 days                               k8s_mysql_wordpress-mysql-b5ddc8dd9-4xmqr_default_7694cbcb-ae3e-4bc8-a767-56c4261202f8_0
340bec496cf7        409c3f937574        "docker-entrypoint.s…"   6 days ago          Up 6 days                               k8s_mongodb_mongodb-5d75bdb4d7-7phxp_default_14986721-98a2-4e22-9e4a-94c569b7702e_0
```

The output of the the `ps` command shows seven columns. Each one gives you basic, but important information, about a container's state.

* `CONTAINER ID` is the unique hash identifier given to a container at runtime. 
* `IMAGE` is the image used to create the container. The image name will also include any tags associated with the image.
* `COMMAND` is the command the container executes at runtime. 
* `CREATED` shows you when the container was originally created.
* `STATUS` shows you how long the container has been running.
* `PORTS` display any network port mappings.
* `NAMES` shows the names each container was given, which like the `CONTAINER ID` also uniquely identifies them. This is a more human-readable value that is easier to remember.


## Listing All Containers
The `docker ps` command alone will only show you running containers. However, it is usually important to list stopped containers as well. This is usually the case when troubleshooting a failed container.
To list all containers, running and stopped, use the `-a` or `--all` flags with the `docker ps` command.

```shell
docker ps --all
```

## Starting Containers

## Stopping Containers

## Deleting Containers
