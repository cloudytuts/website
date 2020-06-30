---
layout: tutorial
title: "Storing Secrets"
date: 2020-06-30T10:55:39-04:00
draft: false
categories: devops
tags: secrets
series: 
weight: 20
author: serainville
abstract: |
    Storing secrets is key to a secure deployment. In this post you will learn how to
    use secretes with your deployments. From deployment to deployment store your secretse safely
    for retrievial.
---

Kubernetes Secrets allow sensitive information to be separated from applications. Isolating area of concerns, an operational team could own the sensitive secrets data, while a development team has full control over thier deployment.

## Concerns
Sensitive information should never be stored in your source code or container images. Separation is key to ensuring the security of your service. 

In this tutorial you will learn:
* How to write a secrets manifest
* How to store binary secrets

## Secrets Manifest
Secrets can be defined as a manifest file using key-pair values.
The following YAML is an example of a secrets manifest.

```yaml
apiVersion: v1
kind: Secrets
data:
  dbpassword: abc123
```