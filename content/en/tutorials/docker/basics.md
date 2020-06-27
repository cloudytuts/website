---
title: "Basics"
weight: 11
categories: ["devops", "containers"]
tags: ["docker"]
---

Learning the basics of Docker is foundamental to using it in your environment. This tutorial will focus on the basic knowledge required to run Docker.

# Running Your First Container

Let's begin by running a simple `hello-world` container. It's a very basic container available from Dockerhub, the default image repository for Docker, that simply outputs a string of texts saying "hello, world!".

```shell
docker run hello-world
```

Once executed Docker will check if the `hello-world` image is available locally. If not the image will be downloaded from Dockerhub. If Docker was installed succesfully and image executed correctly, the following message will be outputted to your screen.

```shell
hello world
your docker installation is working.
```

Most containers are expected to be long running services. However, some like the `hello-world` container simply execute and exit. The `docker ps` command outputs a list of all running containers.

```shell
docker ps
```

Notice how it outputs an empty list. In order to view previously executed container you will need to use the `docker ps -a` command.


# Checking Running Containers



