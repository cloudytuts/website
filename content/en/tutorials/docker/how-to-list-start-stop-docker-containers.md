---
title: "How to List, Start, and Stop Docker Containers"
date: 2020-08-29T23:24:19-04:00
draft: true
author: serainville
tags:
  - docker
description: |
    Learn how to list running containers, as well as how to start, stop, and delete containers using the docker command.
---

In this tutorial, you will learn how to list, start, stop, and delete Docker containers.

With the start command you will learn how to restart a stopped container or multiple stopped containers. By doing so we are able to run them as if they were never stopped, meaning the same volumes will be attached as well as the same port maps.

With the stop command you will learn how to stop running containers. This command can target a single container or multiple containers.

The list command is usually to show you what is running on your host. It gives important information, such as the ID, Name, and basic state of each container running. It call also be used to list stopped containers, which is needed when troubleshooting problems.


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
A stopped container can be restarted using the `docker start` command. This is command is different from the `docker run` command, as the latter creates a new container, while the former starts an existing container that has stopped.

You will need to the `container id` or the `name` to restart a container. This information is easily available from the `docker ps -a` command.

```shell
docker run <container>
```

An interactive session can be created with a restarted container, attaching the container's STDIN to your shell. This is done using the `-i` or `--interactive` flags with the `docker start` command. This is a usefully way of troubleshooting a filed container or auditing its files, for example.

```shell
docker start -i <container>
```

## Restarting Multiple Containers
The `docker start` command is capable of starting multiple containers. If you have multiple stopped containers that you want to run again, you can list them all in the command.

```shell
docker start <container-1> <container-2> <container-3> ...
```

For example, to start containers with IDs `e51898f31223`, `bc97fe8bcd66`, and `17714162a329`, run the following command.
```shell
docker start e51898f31223 bc97fe8bcd66 17714162a329
```


## Stopping Containers
Whenever you need to stop a container the `docker stop` command is used. This command is used against a `container id` or a `name` and it stops the specificed container.

```shell
docker stop <container>
```

For example, to stop a container with ID 17714162a329, run the following command.

```shell
docker stop 17714162a329
```

## Deleting Containers
Finally, to delete a stopped container from your Docker host you use the `docker rm` command. The container must be stopped for the command to work, otherwise it will fail. 

```shell
docker rm <container>
```

If a volume is attached to the container you want to delete, it can be deleted with it by using the `-v` or `--volumes` flags.

```shell
docker rm -v <container>
```

## Deleting Multiple Containers
Like starting and stopping, you can delete multiple containers by listing their IDs or names with the `docker stop` command.

```shell
docker rm <container-1> <container-2> <container-3>
```


