---
title: "How to Effectively use Kubernetes Quality of Service"
date: 2021-10-06T16:51:23-04:00
draft: false
author: serainville
tags: 
  - kubernetes
description: |
    Learn how to leverage Kubernete's Quality Of Service profiles to better control pod evications and resource utilization.
---

[Quality of Service](https://kubernetes.io/docs/tasks/configure-pod-container/quality-service-pod/#create-a-pod-that-gets-assigned-a-qos-class-of-guaranteed) is a feature [introduced in Kubernetes 1.22](https://kubernetes.io/blog/2021/08/04/kubernetes-1-22-release-announcement/) that allows resource profiles to be applied to pods. As of this release there are three profiles available:

- Guaranteed
- Burstable
- BestEffort

Profiles are applied to pods based on how they are configured, and each profile has its own strength and weaknesses you will need to understand to fully leaverage the capabilities they offer.


## Quality of Service Profiles
### Guaranteed
The Guaranteed QoS profile ensure that a pod is scheduled on a node with enough resources to run. The profile is applied to a pod when the request values and limit values are equal.

**Benefits:**
This profile quarantees a pod will have sufficient memory and CPU on any node it is scheduled on; There is no risk of a scheduled pod experiencing resource contention.

**Risks:**
Pods with this profile will only be scheduled when a node has sufficient resources available. If no nodes in your node pools have enough resources the pod will not be scheduled, potentially leading to a service outage.

**Pod that applies the Guaranteed QoS profile:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: prod
spec:
  containers:
  - name: qos-demo-4-ctr-1
    image: nginx
    resources:
      limits:
        memory: "200Mi"
      requests:
        memory: "200Mi"
```

### Burstable
The Burstable QoS profile allows a pod to consume more resources than are requested. However, when being scheduled there is no quarantee the node will have enough resources available for its containers to run.

The profile is assigned to pods where the resource limit values do not match the resource request values. During scheduling, a pod will be placed on a node that meets its request values, however, there is no guarantee the node will have enough resources to meet the resource limit values.

**Benefits:**
The profile allows pods to consume more resources than they request. This extends their range of nodes they can be scheduled on without reserving more memory than is typically needed.

**Risks:**
- **Resource Contention:** When using burstable profiles there is risk of resource contention on a node, as there is no guarantee the node has enough resources to offer a pod.

- **Evication:** When deciding which pods to evict when a node is under pressure, pods with resource requests values that do not equal their resource limit values.

**Pod that applies the Burstable QoS profile:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: prod
spec:
  containers:
  - name: qos-demo-4-ctr-1
    image: nginx
    resources:
      limits:
        memory: "400Mi"
      requests:
        memory: "200Mi"
```


### BestEffort
The BestEffort policy allows for pods to be scheduled on any node, whether there are enough resources to run its applications or not.

**Benefits**
This profile allows for the largest range of node pools a pod can be scheduled on. It is best used for apps or services that are not sensitive to evictions.

**Risks:**
- **Resource Contention:** Pods running with the BestEffort policy have no set resource limits set. Therefore, they are capable of consuming a significant amount of resources, starving other pods.

- **Evication:** Pods with the this profile are among the first to be evicted when a node experiences resource presures.

**Pod that applies the BestEffort QoS Profile:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: prod
spec:
  containers:
  - name: qos-demo-4-ctr-1
    image: nginx
```

## Conclusion
Kubernetes new Quality of Service features grants operators and application owners better control over service quality and pod evication. 

By applying an effective strategy with QoS profiles you can quarantee resources for your highest priority pods, while providing best effort for those that are not as sensitive to evictions or resource contention.


