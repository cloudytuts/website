---
title: "How to Restarting Kubernetes Pods"
date: 2020-08-24T01:25:19-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - pods
description: |
    Learn how to restart your running pods with the kubectl command.
---

In this tutorial, you will learn different methods for forcing pods in a deployment to restart. 

Not all application failures will result in a container or pod from exiting. For scenarious like these you will need to manually restart your Kubernetes Pod. 

## Rolling Restart
A rolling restart can be used to restart all pods from deployment in sequence. This is the most recommend stragety as it will not result in a service outage. 

Rolling restarts were introduced in Kubernetes 1.15. In order to use it both your cluster and your `kubectl` installation must be vsersion 1.15 or higher.

The syntax for rolling restart is the following.

```shell
kubectl rollout restart deployment <name>
```

For example, to perform a rolling restarting on a deployment named `wordpress`, execute the following command:

```shell
kubectl rollout restart deployment wordpress
```

## Changing Replicas
Another strategy for restarting Pods is to scale the number of deployment replicas to zero, and then scaling back up to the desired state. This forces all current pods to stop and terminate and then for new pods to be scheduled in their place.

Be warned. Setting the number of replicas to zero will cause an outage and a rolling restart is the recommended approach.

To set a deployments replicas to zero you would use the following command.

```shell
kubectl scale deployment <name> --replicas=<num>
```

For example, to scale a deployment named `mongodb` to 0 replicas, you would execute the following command.

```shell
kubectl scale deployment mongodb --replicas=0
```

Once all of the pods have been terminated you can scale the deployment back up to its desired state.

```shell
kubectl scale deployment mongodb --replicas=3
```

## Stopping Pods
The third and least practical example is to stop each service manually. The deployment will notice the state change and begin rescheduling new pods until the desired number of replicas is achieved again.

For example, print a list of servers that match a particulr label.
```shell
kubectl get pods -l myapp
```

For each result, delete the pod manually using the `kubectl` command-line tool.

```shell
kubectl delete pod <name>
```