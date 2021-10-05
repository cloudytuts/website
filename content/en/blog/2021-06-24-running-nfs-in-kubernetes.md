---
title: "Running Nfs in Kubernetes"
date: 2021-06-24T22:02:04-04:00
draft: false
author: serainville
description: Is running an NFS server in your Kubernetes cluster a good idea or is it a gateway for hackers
---

One of the many benefits of a network file system is its capability to perform multi-read-write. And as with everything these days, an NFS is just another service that one can run inside of their Kubernetes cluster.

However, is an NFS server an appropriate service to run in a containerized environment?

It may seem innocent enough to run, just as you would many other types of services. The CNCF itself evens promotes the idea via tutorials available within their own documentation, so one would assume the practice is acceptable.

To understand the security implications of running a service like NFS from within your cluster you need to understand how an NFS server runs.

## Container Breakouts
Before we can answer the question of whether NFS is safe to run in your cluster, you should understand what a container breakout is and what is required for it to happen.

A container breakout is the holy grail of any exploited container. It permits a malcious user access to node host and potentially to all other workloads running on the cluster. It may also permit the attacker to compromise other nodes.

In order for a breakout to occur the following must be true:
1. A container running as root (UID 0) or with unsafe kernel capabilities.
2. A container running as privilged

Containers employ namespaces as a means to isolate processes from one another. The primary purpose is to provide security that prevents processes from conflicting with each other.

Unfortunately, many consider that security as security against hackers and vulnerability exploitations. While, yes, it does provides SOME layer of protection, the namespace kernel feature was not designed with that in mind.



## Running NFS
An NFS server require root privileges to run, and the reasons why are plent.

First, an NFS server must be able to bind to a network port in order for clients to connect. Binding to ports requires root privileges.

Second, a number of syscalls are used to host network storage that also require root privileges.

## Kernel Capabilities
Giving anyone or service full root privileges is often thought of as a really, really bad idea. And before kernel capabilities were introduced in the 2000s, this was a common practice when a process required elevated privileges.

Now a days we are able to carve out distinct privileges via kernel capabilities to grant

## Root
NFS must run as root or as user with root privileges. 

## Putting It All Together
Container breaks are the holy grail of hacking any organizations. When accomplished a malcious actor i

* Parent process UID 0 (Root)
* Root privileges


