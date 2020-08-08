---
title: Continuous Delivery with Ansible and Docker
draft: false
author: serainville
date: "2020-08-07"
description: |
    Learn how to use Ansible and Docker to implement an effectively Continuous Delivery strategy in your environment, and accelerate your production releases.
tags:
  - Ansible
  - Docker
  - Jenkins
---

Continuous delivery has empowered today's most succesful tech companies and it is a key ingredient to highly effective DevOps cultures. In this guide you will learn how to implement a continuous delivery solution using Ansible and Docker.

## Getting Started
In order to follow along with this guide you will need the following installed on your local development machine.
- Docker
- Ansible

You should also have the following on a separate macine.

### Running Jenkins as a Container
For the purpose of this guide a Jenkins server will be deployed using Docker. In a typical environment your build server would be hosted on a separate server or compute instance. 

```shell
docker pull jenkins\jenkins
```

Start a development version of Jenkins in a container from the image just pulled down.

```shell
docker run -d -p 8080:8080 -v /usr/jenkins/workspace jenkins\jenkins
```
