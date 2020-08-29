---
title: "How to Remove Docker Containers and Volumes"
date: 2020-08-28T21:39:44-04:00
draft: false
author: serainville
tags:
  - docker
  - volumes
  - containers
description: |
    Learn how to delete old and unsused Docker containers and volumes and reclaim free space on your host
---

In this tutorial, you will learn how to remove Docker containers and volumes from your Docker host. This information is especially useful for anyone running a docker development environment, where frequent image builds and runs are not uncommon.

Docker images and containers consist of several container layers, one for each action in a Dockerfile. As we change actions old container layers are orphaned and replaced by new container layers. As time goes on these orphaned containers eat up unnecessary disk space that can easily be reclaimed. 

The reason old container layers are left on disk is for cache reasons. Each container is given a unique hash, and any new actions that mirror that already creater layer use that layer instead of creating a new one. This significanlty speeds up the Docker build processes. It also leaves a lot of cruft behind until those layers are manually removed.

Volumes are similar to container layers in that they are only removed manually. There is not automated process to clean up old, unused volumes. 

## Remove Containers
To clean up old containers from a Docker host you use the `docker rm` command. The command takes a name or ID of a container on the system.

To output a list of all containers execute the `docker ps`. However, the command alone will only list running containers. To list stopped containers too you use the `-a` flag.

```shell
docker ps -a
```
```shell
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
cf332ff54f7b        wordpress           "docker-entrypoint.s…"   About a minute ago   Exited (0) 8 seconds ago                       gracious_golick
87d2f748880a        wordpress           "docker-entrypoint.s…"   About a minute ago   Exited (0) 2 seconds ago                       modest_galois
```

To delete a container we use the `docker rm` command. The command takes a container name its ID as a parameter, and multiple containers can be remove at once.

### Deleting one container
To remove a single WordPress container from the list above, the following command is executed.
```shell
docker rm cf332ff54f7b
```

### Deleting Multiple Containers
To remove both containers, we would run the following command.
```shell
docker rm cf332ff54f7b 87d2f748880a
```

### Delete Volumes too
If your old containers had volumes attached to them, these old, orphaned volumes can be removed with the container. To instruct the `docker rm` command to include volumes when deleting containers, include the `-v` or `--volumes` flag.

For example, to delete a single container and its volumes run following command.
```shell
docker rm -v cf332ff54f7b
```

To delete multiple containers and their volumes, use the following command.
```shell
docker rm  -v cf332ff54f7b 87d2f748880a
```

### Delete all stopped containers
Manually removing a large list of containers using the `docker rm` command is cumbersome. When dealing with a high volume of containers a better strategy is needed. In the case of deleting all stopped containers, we use the `docker container prune` command.

```shell
docker container prune
```

You will be prompted to confirm this action.
```shell
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] 
```

If successfully, a list of container IDs for all the containers that have been removed will be printed to the screen.

```shell
Deleted Containers:
f53496dff1845f2dc8c544dcf4f6a61c9b057db0dfca8f356c3b9759b6a05a24
f907afa9755dfde89e28cb8e9ef37b29985ea76c7862940d60e0317080f0da40
5fd4589bc1a10db918f0e0193f485484ce19659dae4f830172780d357f679744
2c477062baea9f4c6ffce618e75a52879298ae4302093990ad951410f0917236
b18c15adcc4a5af24eb08a96852a533d988a272a35ef26a16cbcc2f05d4719db
a8a8ea8c8357cd592989cee8dd5218d8879433b47ede2d2610f587f3ea32a308
e9c96adfb08eaebf608878470cb29d8d51716526041aecd00df0e38631931334
583b7ef4cb93ca39a32dc7be2486467027883a923495f16af0adaeb9e3fc8296
6bc0ab3c4ed0ac4735a67edc54ed98e8d1342709344364b8f01e846abbfd896e
699b05eda7d82cbed524ac428112f38405f4d4631ededd30ec589b6f7328ee36
dc1891884e3f55c8ceb0fe6e552c72919d84d9cf09fa8fa8c1e2405b63d4f2b5
73a95303a56ca19c16303fcd0c9f2256d595141ceb7b93b6c189092448b2af27
cf332ff54f7be8586392bfeb79117b2db784e499edf7eedb5cb2c48c1ceabc09
53f9e110832393d2531705e05594a1d0c1103e363f8a5c3c7b73cb0e09d5b8e5
87d2f748880a165a95a9dbe08f52def66076afa2c53c2cc95df74e877d7e486d
5542407bf60b8b7dbc288baec1acc89991d21148203b43d79fa42ff4d5d9f1e8

Total reclaimed space: 4.272kB
```

The output also include the amount of storage reclaimed during the pruning process.

## Deleting Volumes
Volumes, like containers, can eat up storage on your Docker host. Orphaned and unneeded containers can take up a lot of storage if left behind. So, as with containers, it is a good idea to regularly clean up your volumes. This is especially true with docker development environments.

To list the volumes on a host use the `docker volume ls` command.

```shell
docker volume ls
```
```shell
DRIVER              VOLUME NAME
local               2cd155e718050ce1716c5606e6c3020e9d9554c52cb1e7b822bb469e8c973972
local               2dea7f06e5a98e25e74ab70068e3930e5695574345c3552162eecf6d86144f32
local               4d4a7782fca5a4dcc3c73587305556d4cacfaa367cab2deefaf4abd38893d617
local               5d75d998becf5081ef6b563432fa3a15a73921de76caf84969ba630a68620e65
local               7d913c4d8c983dd97dbdb4a900530ef9dc6187f1420e0ec6cc30f26d66c4ce9f
local               9dc743679fbdf9dfb42265cccab305985e1581131cbbe9cf0bdf217cf105361f
local               9f10581b40a0a9aa365c8bba252a6c169d85e893ff8b7230fbe3443cbf1af5d6
local               48aab0d5380d189b193225e69143521e3d3aa6299d323cc64bd7e5359936ed60
local               62d97f62b383940af0c3ca728621317351edc45fbef3aa19163647cbe1ee3708
local               435f5be4efd7fe3517bd2313c9f7d51c171fe49afb455c8001f3ce6ddfc4d06b
local               888ebcf26e9fbddbba70bd8a4b12e6d45896bcedef8e938bb614c5eb058baafa
local               3119c974641534a87ffc4122c3afb6270267539eb1aa26de1dde179ee3441074
local               13640c9d79ed6060ede836966a4f5661d3cfb37ad5a185db6ae7493e7731ad95
local               c54a3cc74ea5f1545e143f8ed6483e983c9d6b7b0013932fa603da591518673f
```

To delete a volume you will need its unique volume hash ID, convienently outputted by the `docker volume ls` command. The command to delete volumes is `docker volume rm`.

```shell
docker volume rm 2cd155e718050ce1716c5606e6c3020e9d9554c52cb1e7b822bb469e8c973972
```

### Delete all unused volumes
Having to manually discover and delete all unused volumes is a painstaking effort. We can achieve the same results, quicker, by using the `docker volume prune` command. Like its container counterpart, you will be prompted to confirm your actions.

```shell
docker volume prune
```
```shell
WARNING! This will remove all local volumes not used by at least one container.
Are you sure you want to continue? [y/N] 
```
```shell
Deleted Volumes:
2cd155e718050ce1716c5606e6c3020e9d9554c52cb1e7b822bb469e8c973972
3119c974641534a87ffc4122c3afb6270267539eb1aa26de1dde179ee3441074
4d4a7782fca5a4dcc3c73587305556d4cacfaa367cab2deefaf4abd38893d617
888ebcf26e9fbddbba70bd8a4b12e6d45896bcedef8e938bb614c5eb058baafa
2dea7f06e5a98e25e74ab70068e3930e5695574345c3552162eecf6d86144f32
7d913c4d8c983dd97dbdb4a900530ef9dc6187f1420e0ec6cc30f26d66c4ce9f
5d75d998becf5081ef6b563432fa3a15a73921de76caf84969ba630a68620e65
13640c9d79ed6060ede836966a4f5661d3cfb37ad5a185db6ae7493e7731ad95
9f10581b40a0a9aa365c8bba252a6c169d85e893ff8b7230fbe3443cbf1af5d6
c54a3cc74ea5f1545e143f8ed6483e983c9d6b7b0013932fa603da591518673f
48aab0d5380d189b193225e69143521e3d3aa6299d323cc64bd7e5359936ed60

Total reclaimed space: 95.46MB
```

## Conclusion
There are number of ways of deleting Docker resources from the command-line. In this tutorial, you learned how to delete containers and volumes from your Docker host, and recovered storage in the process.