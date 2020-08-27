---
title: "How to Add Entire Directory of Files to a ConfigMap"
date: 2020-08-26T23:47:12-04:00
draft: false
author: serainville
tags:
  - kubernetes
  - configmaps
description: |
    Learn how to add an entire directory of files to your ConfigMap for mounting in a deployment or pod.
---

In this tutorial, you will learn how to store an entire directory of files in ConfigMap, without adding them individually. 

Most use cases of ConfigMap are to store parameters for an application. On occasion, individual files are stored for mounting in a deployment or pod. These files are typically single config files, such as `my.conf` or `mongod.conf`.

However, when your use case requires an entire directory of files to added to a ConfigMap, rather then add themes files individually we can add the entire directory.

## Create ConfigMap
When we want to add a file to a ConfigMap we use the `--from-file` flag with the `kubectl create configmap` command. The most common use case for `--from-file` is adding individual files, but it can also target directories as well.

If the value of the `--from-file` flag is a directory, kubectl will scan the directory and add each file as a key-value pair in the ConfigMap.

```shell
kubectl create configmap config-files --from-file=/etc/configs
```

To verify each file in the directory was added as key-value pair in the ConfigMap you can `kubectl get` it. Output the queried resource in YAML to verify each file was added with its contents.

```shell
kubectl get configmap config-files -o yaml
```
```yaml
apiVersion: v1
data:
  client.conf: |
    parameter2=test
    parameter3=test
  routes.conf: |
    parameter4=http://www.example.com
  server.conf: |
    parameter1=hello
kind: ConfigMap
metadata:
  creationTimestamp: "2020-08-27T03:56:54Z"
  name: config-files
  namespace: default
  resourceVersion: "536004"
  selfLink: /api/v1/namespaces/default/configmaps/config-files
  uid: 314a5692-987e-4aed-97cf-77615e7812e3
```

We can see three keys, one for each file that was scanned. The files were `client.conf`, `routes.conf`, and `server.conf`. Each file's contents can also be seen.

{{< warning >}}
ConfigMap have a hard 1MB size limit. If the contents of your files exceeds 1MB you will not be able to store them in a single ConfigMap. A Persistent Volume may be a better solution.
{{< /warning >}}

## Conclusion
While most ConfigMap use cases are for storing parameters or a few individual files, there are times when an entire directory of files must be added. In this tutorial, you learned that the `--from-file` flag for the `kubectl create configmap` command can do more than just add single files, it can add an entire directory of files.


