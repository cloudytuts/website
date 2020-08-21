---
title: "How to List Running Container Image Versions in Kubernetes"
date: 2020-08-21T13:48:34-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - version-checker
    - kubectl
description: |
    Learn how to kubectl or an opensource utility named version-checker to list all version of running images in your Kuberenetes cluster.
---

Keeping your container images compliant in a production environment is important for the security of your services. While this is not a command available in `kubectl`, we can still use `kubectl` to generate a list of runtime images. 

Altneravily, we can use an opensource utility named `version-checker` instead. The tool runs as a service which continuously monitors the runtime images of your containers, and then outputs a list using Prometheus metrics over TCP port 8080. Even more valuable and not avialable with `kubectl` is the ability to compare current images with upstream versions, allowing you to see how out-of-date your images are.


## Using Kubectl
There is no elequent way of generating a list of running container images in your cluster today. However, it can still be accomplished using `kubectl` with the `-o` flag. 

The following command will find all resources with the `image` path and output the value in each.
```shell
kubectl get pods --all-namespaces -o jsonpath="{..image}" 
```
    | Output
```shell
serverlab/feedorus-api:1.0.6 serverlab/feedorus-api:1.0.6 serverlab/feedorus-api:1.0.6 serverlab/feedorus-api:1.0.6 busybox:1.32 mysql:5.7.30 busybox:1.32 mysql:5.7.30 serverlab/feedorus-worker:1.0.0 serverlab/feedorus-worker:1.0.0 serverlab/feedorus-worker:1.0.0 serverlab/feedorus-worker:1.0.0 serverlab/feedorus-worker:1.0.0 serverlab/feedorus-worker:1.0.0 feedorus/worker:0.1.15 feedorus/worker:0.1.15 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.32.0 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.32.0 docker.io/cilium/operator:v1.7.5 cilium/operator:v1.7.5 docker.io/cilium/cilium:v1.7.5 docker.io/cilium/cilium-init:2019-04-05 cilium/cilium-init:2019-04-05 cilium/cilium:v1.7.5 docker.io/cilium/cilium:v1.7.5 docker.io/cilium/cilium-init:2019-04-05 cilium/cilium:v1.7.5 cilium/cilium-init:2019-04-05 docker.io/coredns/coredns:1.6.7 coredns/coredns:1.6.7 docker.io/coredns/coredns:1.6.7 coredns/coredns:1.6.7 quay.io/k8scsi/csi-node-driver-registrar:v1.2.0 digitalocean/do-csi-plugin:v2.0.0 digitalocean/do-csi-plugin:v2.0.0 quay.io/k8scsi/csi-node-driver-registrar:v1.2.0 quay.io/k8scsi/csi-node-driver-registrar:v1.2.0 digitalocean/do-csi-plugin:v2.0.0 digitalocean/do-csi-plugin:v2.0.0 quay.io/k8scsi/csi-node-driver-registrar:v1.2.0 docker.io/digitalocean/do-agent:3 digitalocean/do-agent:3 docker.io/digitalocean/do-agent:3 digitalocean/do-agent:3 gcr.io/google-containers/hyperkube:v1.18.6 docker.io/library/busybox:1.30 gcr.io/google-containers/hyperkube:v1.18.6 busybox:1.30 gcr.io/google-containers/hyperkube:v1.18.6 docker.io/library/busybox:1.30 gcr.io/google-containers/hyperkube:v1.18.6 busybox:1.30 wordpress:5.5.0-php7.2-apache wordpress:5.5.0-php7.2-apache%
```

While this gives us the information we need, it isn't easily read. We can format the list by piping it to various utlities, such as `tr` to trim spaces, `sort` to sort, and `unique` to elminate duplicate lines.

### List All Namespaces
The following command will generate a list of all images running in your Kubernetes cluster. We've added the `-c` flag to the `unique` command to gives a count of each unique image.

```shell
kubectl get pods --all-namespaces -o jsonpath="{..image}" |\
tr -s '[[:space:]]' '\n' |\
sort |\
uniq -c
```
    | Output
```shell
   2 busybox:1.30
   2 busybox:1.32
   2 cilium/cilium-init:2019-04-05
   2 cilium/cilium:v1.7.5
   1 cilium/operator:v1.7.5
   2 coredns/coredns:1.6.7
   2 digitalocean/do-agent:3
   4 digitalocean/do-csi-plugin:v2.0.0
   2 docker.io/cilium/cilium-init:2019-04-05
   2 docker.io/cilium/cilium:v1.7.5
   1 docker.io/cilium/operator:v1.7.5
   2 docker.io/coredns/coredns:1.6.7
   2 docker.io/digitalocean/do-agent:3
   2 docker.io/library/busybox:1.30
   2 feedorus/worker:0.1.15
   4 gcr.io/google-containers/hyperkube:v1.18.6
   2 mysql:5.7.30
   4 quay.io/k8scsi/csi-node-driver-registrar:v1.2.0
   2 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.32.0
   4 serverlab/feedorus-api:1.0.6
   6 serverlab/feedorus-worker:1.0.0
   2 wordpress:5.5.0-php7.2-apache
```

The output is formatted into two columns.
```text
<count> <image>:<tab>
```
The first column shows count of containers using a specific runtime image, while the second column gives you the image name and its tag.

### List by Namespace
To filter the list by namespace you can use the `--namespace` or `-n` flag in your `kubectl` command.

```shell
kubectl get pods -n wordpress-blog -o jsonpath="{..image}" |\
tr -s '[[:space:]]' '\n' |\
sort |\
uniq -c
```

## Using Version-checker
Version checker is an opensource utility by Jetpack that runs in your Kubernetes cluster. The ulitity runs as a service that exposes Prometheus metrics over TCP port 8080, and is intended to be used alongside Prometheus and a dashboard such as Kibana.

### Install using Helm
Version-Checker is available as a Helm chart for ea

```shell
heml install version-checker .
```





