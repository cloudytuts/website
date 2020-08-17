---
title: Continuous Delivery with Ansible and Docker
draft: false
author: serainville
date: "2020-08-07"
description: |
    Learn how to use Ansible and Docker to implement an effectively Continuous Delivery strategy in your environment, and accelerate your production releases.
category:
  - CICD
tags:
  - Ansible
  - Docker
  - Jenkins
  - Go
---

Continuous Delivery is the ability to continuously delivery your application into a target environment. 
Continuous delivery has empowered today's most succesful tech companies and it is a key ingredient to highly effective DevOps cultures. In this guide you will learn how to implement a continuous delivery solution using Ansible and Docker.

## Getting Started
In order to follow along with this guide you will need the following installed on your local development machine.
- Docker
- Ansible

You should also have the following on a separate macine.




Docker images should be as lean as possible

commit -> dockerfile --> push dockerhub

## Workflow
- Build Docker Image
- Test Docker Image
- Release Docker Image


## Test
### Integration Tests using Docker
Integration tests are important for testing new feature merges against components of your application. Our pipeline will use

```groovy
stage("IST") {
  sh: go test
}
```

## Build
We provide the build

```shell
docker build -t todobackend-dev -f dockerfile/dev/Dockerfile .
```


```shell
docker run -v ./src:/src golang:1.14.7 go test /src/...
```

```dockerfile
FROM golang:1.14.7 AS build
```


### Base Docker Images
When containerizing an application a base image must be chosen. There are plenty of base images created and maintained by the community available. In the past it was common to use treat containers like an operating system, and as such, base images of operating systems were often used. Over the years this approach has fallen out of favour, due to security and storage efficiency. A bsae image based on a popular distribution of Linux, such as Ubuntu, will have many packages that are not required to run most applications, while creating a much larger security footprint that must be protected.

Light-weight images strongly recommended as base. Alpine Linux is an ultra light-weight distribution of Linux and is one of the most popular for base images. So popular that third-party images typically have an Alpine release.

### Multistage Docker Builds
When working with applications that must be compiled prior to being released a single stage docker build, the default build strategy, is a poor choice. With a single stage build your image will contain more than just the product of the compile, it will also include the required build tools and source files.

```dockerfile
FROM golang:1.13 AS build

COPY ./src ./

RUN go test
```


## Release

## Deploy

Once you've successfully built a release-ready Docker image you are ready to move onto the deployment phase. 

### Ansible Role

```yaml
---
tasks:
- name: Deploy MyAPP
  command: docker -h ${docker_host} run myapp:${release_version}
```

### Running Jenkins as a Container
For the purpose of this guide a Jenkins server will be deployed using Docker. In a typical environment your build server would be hosted on a separate server or compute instance. 

```shell
docker pull jenkins\jenkins
```

Start a development version of Jenkins in a container from the image just pulled down.

```shell
docker run -d -p 8080:8080 -v /usr/jenkins/workspace jenkins\jenkins
```

