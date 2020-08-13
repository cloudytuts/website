---
title: "Securely Store TLS Certificates as Kubernetes Secrets"
date: 2020-08-13T12:09:15-04:00
draft: false
author: serainville
tags:
    - certificates
    - security
    - kubernetes
    - secrets
description: |
    Learn how to securely store your your application's TLS certificate key-pairs in Kubernetes using secrets
---

Certificates provide a means of securing communication on the Internet. 

In order to store TLS certificates in Kubernetes a public/private key pair must exist. The public key certificate must be .PEM encoded and match the given private key. 

## Creating a TLS Secret
The `kubectl` CLI provides a command to easily store TLS certificate key-pairs in Kubernetes as secrets. 

```shell
kubectl create secret tls <SECRET-NAME> --cert=<PATH/TO/CERT/FILE> --key=<PATH/TO/KEY/FILE>
```

For example, to create a secret name `webapp-tls-production` in Kubernetes with a public\private key pair, you would execute the following command.

```shell
kubectl create secret tls webapp-tls-production --cert=webapp.pem --key=webapp.key
```

### Dryrun

The `kubectl` command provides a way to perform a dryrun of the `kubectl create secret` commad. Use this as away to verify your secret is created correctly and minimize errors.

```shell
kubectl create secret tls webapp-tls-production --cert=webapp.pem --key=webapp.key
```

## Manifest File
Manifest files can also be used to create TLS secrets in Kubernetes. 

In order to correctly store TLS key-pairs in Kubernetes as a secret, you must do the following in your manifest file:
* Set `type` to `kubernetes.io/tls`
* Base64 encode contents of your key-pair files, and add them as `data` keys: `tls.crt` and `tls.key`

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: webapp-tls-production
type: kubernetes.io/tls
data:
  tls.crt: --BASE64 ENCODED STRING--
  tls.key: --BASE64 ENCODED STRING--
```
