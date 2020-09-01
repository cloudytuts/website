---
title: "How to Create a Docker Pipeline With Jenkins"
date: 2020-08-30T23:57:01-04:00
draft: false
author: serainville
tags:
  - jenkins
  - docker
  - cicd
description: |
    Learn how to create a Docker pipeline with Jenkins to build and run containers on a remote host
---

In this tutorial, you will learn how to create a Docker pipeline with Jenkins that will build, deploy, and run containers on remote hosts. 

## What You will Learn
* Executing Docker commands on remote hosts
* Creating a Docker pipeline job in Jenkins

## Prerequisites
In order to create a Docker CICD pipeline in Jenkins, you will need the following.
* A Jenkins instance
* Docker installed on the Jenkins node

## Dockerfile


## Remote Docker Commands
Docker supports executing commands on a remote host, which will be key for creating a continuous delivery pipeline. The connection type used will be SSH for increased security.

To execute a remote command with Docker you use the `-H` or `--host` flag. 

```shell
docker --host ssh://user@host [options] [commands]
```
### Jenkins Credential
The best method for accessing a host remote is by using SSH with a certificate key-pair. The key pair should be used specifically for remote Docker executions, and it should have a passphrase for extra security.

Create a new key-pair for SSH authentication.

```shell
ssh-keygen
```
You will be prompted to enter the path and filename of the generated key files, and then for a passphrase. The following output shows a new key named `jenkins_d_rsa`. 
```shell
Enter file in which to save the key (/Users/devops1/.ssh/id_rsa): jenkins_id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in jenkins_id_rsa.
Your public key has been saved in jenkins_id_rsa.pub.
The key fingerprint is:
SHA256:ugBK0kO/NJkpS6ePZpSMeyHK7fT8dlaiQz81DLyxrmo devops1@Devops-MacBook-Pro.local
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|                 |
|  .     .        |
| o . +   +       |
|.+*.O   S *      |
|=o=O o o + =     |
|+++oo o + + .    |
|o.=+oE = *       |
| +o.o+=o= .      |
+----[SHA256]-----+
```




## Docker Pipeline
The Docker pipeline in this example is fairly basic. We will begin by building an a new Docker image for our application, which will use a few built-in Jenkins variables for versioning. 

Once built, the new Docker image will be deployed to our image repository. This provides as an artifact to deliver into any environment, from staging to production. 

```groovy
node {
    stage("Build") {
        sh "docker build -t myapp:${version} ."
    }
    stage("Deploy Artifact") {
        sh "docker push -t myapp:${version}"
    }
    stage("Deploy Stagin"){
        sh "docker -h ssh://jenkins@10.0.0.10 run -d -p 80:80 myapp:${version}"
    }
    stage("Deploy Prod"){
        sh "docker -h ssh://jenkins@10.0.1.10 run -d -p 80:80 myapp:${version}"
    }
}
```