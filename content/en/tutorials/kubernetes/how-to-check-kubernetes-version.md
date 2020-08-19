---
title: "How to Check Kubernetes Version"
date: 2020-08-19T10:07:13-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - kubectl
description: |
    Learn how to use the kubectl cli to check which version of Kubernetes your clusters are running, and the version of Kubernetes each node is running.
---


## Cluster Version
The simplest way of checking a cluster's Kubernetes version is to use the `kubectl version` command. This command will output information for the `kubectl` client and the Kubernetes cluster.

```shell
kubectl version
```
    | Output
```shell
Client Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.8", GitCommit:"9f2892aab98fe339f3bd70e3c470144299398ace", GitTreeState:"clean", BuildDate:"2020-08-14T11:09:22Z", GoVersion:"go1.14.7", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"18", GitVersion:"v1.18.6", GitCommit:"dff82dc0de47299ab66c83c626e08b245ab19037", GitTreeState:"clean", BuildDate:"2020-07-15T16:51:04Z", GoVersion:"go1.13.9", Compiler:"gc", Platform:"linux/amd64"}
```

The *Server Version* is the version of Kubernetes your cluster is running. You can the Major version, the minor version, as well as the Git version. The latter includes the bug fix.

In the example above, we can see that the Kubernetes cluster the command was ran against is running version 1.18.6. We can also see that our client is ahead of the cluster with version 1.18.8. 

## Node Version
To check which version each node is running we use the `kubectl get nodes` command. The output will list all of a cluster's nodes and the version of Kubernetes each one is running.

```shell
kubectl get nodes
```
    | Output
```shell
NAME                   STATUS   ROLES    AGE   VERSION
pool-am5ukodj8-3j49o   Ready    <none>   26d   v1.18.6
pool-am5ukodj8-3j49x   Ready    <none>   26d   v1.18.6
```

