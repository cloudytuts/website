---
title: "How to set Default Kubernetes Namespace"
date: 2020-08-17T14:18:50-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - namespace
description: |
    Learn how to set a default namespace using the kubectl command instead of having to specify it in commonly used spaces.
---

In this tutorial, you will learn how to set a default namespace context for kubectl.

Kubernetes organizes all of its resources into namespaces. As an organizational unit, namespaces provide a means to apply separation of concern for resources used by different projects or teams, for example.

By default all resources are created in the `default` namespace, unless a namespace is specified. This is also true for all commands against your cluster. When a namespace is not specified in a `kubectl` command with the `-n` flag, Kubernetes will return results from resources in the `default` namespace.

## Setting Default Namespace
Namespace defaults are set in your cluster's context configuration. We change the default you will need to use the `kubectl` `set-config` command and specify the name of the namespace want to be used as default.

```shell
kubectl config set-context --current --namespace=NAMESPACE
```

For example, to set the namespace `team-a` as your default, you would run the following command:
```shell
kubectl config set-context --current --namespace=team-a
```


## Using Alias
Setting a default namespace helps cut down your command when working in a common space. Alternatively, if you find yourself working a number of namespaces you could create a bash `alias` instead. an `alias` could be created for each namespace you work with.

The `alias` command syntax is as follows. You simply give the alias a name and set the command to be executed. An example of a `kubectl` alias is shwon below.
```shell
alias <ALIAS>='kubectl -n <NAMESPACE>'
```

For example, to create an alias for a namespace named `team-a` you would use the following command.
```shell
alias team-a=`kubectl -n team-a`
```

With the `alias` set, to perform `kubectl` actions against a your cluster and desired namespace you would execute a command using the alias, in addtion to which ever commands or flags you want passed to `kubectl`. 

To use an alias to output a list of pods for the aliased namespace `team-a`, you would execute the following commmand.

```shell
team-a get pods
```
