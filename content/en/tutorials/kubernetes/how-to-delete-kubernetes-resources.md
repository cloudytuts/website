---
title: "How to Delete Kubernetes Resources"
date: 2020-08-21T10:25:59-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - namespaces
    - pods
    - services
    - deployments
description: |
    Learn how to delete Kubernetes resources by using a manifest file or by targeting the resource directly through the command-line.
---

# What's Covered
* Learn how to delete Kubernetes resources using manifest files.
* Learn how to delete Kubernetes resources without manifest files.

## Deleting Resource using Manifests
The simplest method of deleting any resource in Kubernetes is to use the specific manifest file used to create it. With the manifest file on hand we can use the `kubectl delete` command with the `-f` flag. 

The manifest file contains all of the information to target a specifc resource. We do not need to specify any other information, such as namespace or label.

**Syntax**
```shell
kubectl delete -f <path/to/file>
```

**Example**
```shell
kubectl delete -f namespace.yaml
```



## Deleting Resources
A manifest file is not required for deleting resources. Instead, the resource can be targetted directly using the `kubectl delete` command.  This method is more effective when targeting a group of resources or for deleting all resources in the cluster or a namespace.

**syntax**
```shell
kubectl delete <type> <name> [-n <namespace>] | --all | -l <label>]
```

### Dry Runs
Deleting resources can be a risky operation. When using complex matches for deleting resources it is a good practice to perform a `dryrun` of the deletion. This will allow you to preview exactly what Kuberentes will delete, ensuring you do not accidently delete something in error.

```shell
kubectl delete ns --all --dry-run
```

## Deleting All Resources
### Current Namespace
To do a mass delete of all resources in your current namespace context, you can execute the `kubectl delete` command with the `-all` flag.

```shell
kubectl delete --all
```

### Specific Namespace
To delete all resources from a specific namespace us the `-n` flag.

```shell
kubectl delete -n wordpress --all
```

### All Namespaces
To delete all resources from all namespaces we can use the `-A` flag.

```shell
kubectl delete -A
```

## Deleting Resource Types
### Deleting Namespaces

**Syntax**
```shell
kubectl delete ns <name>
```

**Example**
```shell
kubectl delete ns myapp
```

### Deleting Pods

**Syntax**
```shell
kubectl delete pod <name> [-n <namespace>]
```

**Example #1**
```shell
kubectl delete pod worker-cgxxv
```

To delete a pod located in a different namespace you must use the `-n` flag with the namespace's name.
**Example #2**
```shell
kubectl delete pod worker-cgxxv -n myapp
```

### Deleting Services
**Syntax**
```shell
kubectl delete svc <name>
```

**Example 1**
```shell
kibectl delete svc nginx
```

**Example 2**
```shell
kubectl delete svc nginx -n wordpress
```

### Deleting Deployments
```shell
kubectl delete deployment <name>
```


