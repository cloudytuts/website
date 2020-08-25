---
title: "Using Hierarchical Namespaces in Kubernetes"
date: 2020-08-15T00:21:35-04:00
draft: false
author: serainville
tags:
  - namespaces
description: |
    Learn how to implement nested namespaces in Kubernetes using the new Hierarchical Namespaces resource type, and move away from two-dimensional resource organization.
---

Namespaces are a way of organizing resources into common groups. The limitation up until now has been the flat, two-dimensional hierarchy of namespaces. 

Unfortunately, at the time of writing this tutorial, hierarchical namespaces are not a built-in feature of Kubernetes or kubectl. Instead, it is available as an extension to be installed on your cluster, which an accompanying plugin for kubectl.

## Install Hierarchical Namespaces
Hierarchical namespaces requires a new controller to be installed on your cluster(s). The controller YAML is available from the [multi-tenancy](https://github.com/kubernetes-sigs/multi-tenancy/releases) repository of Kubernetes' [kubernetes-sigs](https://github.com/kubernetes-sigs) project.

### Install HNC Controller
The new HNC controller can be installed by downloading a YAML file from Kubernetes' SIGs repository for multi-tenancy. To install the controller on your Kubernetes cluster run the following commands. 
```shell
HNC_VERSION=v0.5.1
kubectl apply -f https://github.com/kubernetes-sigs/multi-tenancy/releases/download/hnc-${HNC_VERSION}/hnc-manager.yaml
```

### Kubectl Plugin
In order for kubectl to support the Hierarchical Namespace Controller a plugin must be installed. The plugin is also available from the Multi-tenancy SIGs repository. To download and install the HNC plugin on your workstation, run the following commands.
```shell
HNC_VERSION=v0.5.1
curl -L https://github.com/kubernetes-sigs/multi-tenancy/releases/download/hnc-${HNC_VERSION}/kubectl-hns -o ./kubectl-hns
chmod +x ./kubectl-hns
```

To verify the plugin installed correctly, use the new `kubectl hns` command.

```shell
kubectl hns
```



## Creating Hierarchical Namespaces
The `kubectl` CLI includes a new command named `hns`, or hierarchical Namespace for short. When used against an existing namespace the new hns namespace will be nested underneath it.

For example, the following creates a nested namespace named **production** under an existing namespace named **myapp**.

```shell
kubectl hns create production -n myapp
```

## Hierarchical Namespace Trees
To help engineers find their way through a nest of namespaces, a very helpful `tree` subcommand has been included with the new `hns` command. 

```shell
kubectl hns tree myapp
```
    | Output
```text
myapp
└── production
```

## Creating Hierarchical Namespaces

### Create Parent Namespace
Let's begin by creating a parent namespace. This namespace will be the top-level namespace in our hierarchy. 

```shell
kubectl create namespace wordpress
```

### Create Nested Namespace
To create a child namespace you do not use the `kubectl create namespace` command. Instead, you use the new `kubectl hns` command. For example, to add a child namespace to the `wordpress` namespace created above.

```shell
kubectl hns create production -n wordpress
```

### Namespace Tree
The Hierarchical Namespace plugin provides topology view of your nested namespaces. It includes a `tree` command to output a visual representation of the hierarchy. 

To view a tree of the `wordpress` namespace, use the `kubectl hns tree` command.

```shell
kubectl hns tree wordpress
```

The output of the command will look like the following.

```shell
wordpress
└── production
```