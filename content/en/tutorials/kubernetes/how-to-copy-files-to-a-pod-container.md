---
title: "How to Copy Files to a Pod Container in Kubernetes"
date: 2020-08-27T23:19:43-04:00
draft: false
author: serainville
tags:
  - kubernetes
  - pod
  - container
description: |
    Learn how to copy files from your local machine into a running Pod container using the kubectl command.
---

In this tutorial, you will learn how to use the `kubectl` CLI to copy files and directories from your local machine into a running Pod container.

The `kubectl` CLI has a `cp` command that can be used to copy local files into a Pod. In the case of a Pod with multiple containers, known as sidecars, the `cp` command allows you to specify exact container to copy files in.

## Copying Files
The `kubectl cp` command is very simple to use. The syntax requires a **files source** and **destination**. 
```shell
kubectl cp <file-spec-src>:<file-spec-dest>
```

### Examples
#### Copy from remote pod
The following example will copy files from the `/foo` directory of the pod named `my-pod` in the `default` namespace. The files will be copied to a local folder with the path `/bar`.

```shell
kubectl cp default/my-pod:/foo /bar
```

In this example, files are copied from the `/var/www/html` directory of a pod named `wordpress` in the `production` namespace. The files are copied to a relative path of `./web-backup` on the local host.

```shell
kubectl cp production/wordpress:/var/www/html ./web-backup
```

#### Copy to remote pod
{{< note >}}
Files copied to a pod will not persist beyond the life the pod, unless the files are copied to a persistent volume's mount path.
{{< /note >}}

Copying files to remote pods is the inverse of copying from remote pods. We specify the local path and then the target spec file. For example, to copy the contents of a local `/foo` directory to the `/bar` directory of a pod named `my-pod`, you would use the following command.
```shell
kubectl cp /foo default/my-pod:/bar
```
This second example show us copying files from the relative path `./static-files` to a pod named `webapp` in the `development` namespace. The target directory in the pod is `/var/www/html`.
```shell
kubectl cp ./static-files development/webapp:/var/www/html
```

#### Copy to specific pod container
To copy files to a specific container in a remote pod, you use the `kubectl cp` command with the `-c` or `--container` flag.

For example, to copy the local directory `./foo` into a container named `logger` of a pod named `my-pod`, you would execute the following command.
```shell
kubectl cp ./foo default/my-pod:/bar --container=logger
```

### Copy from a specific pod container
To copy files from a specific container in a remote pod, you would use the `kubectl cp` command with the `-c` or `--container` flag. For example, the following copies from file the `/etc/config.conf` file of a container named `logger` of the pod `mysql`. The files are copied to a local directory path of `/tmp`

```shell
kubectl cp default/mysql:/etc/config.conf /tmp -c logger
```

## Conclusion
Files can easily be copied to and from remote pods of a Kubernetes cluster. This is definitely important for applying quick fixes while troubleshooting issues, or for acquiring files for analysis. 

It's important to note that pods and their containers are ephemeral. Any files copied to a pod or one of its containers will not persist beyond the life of the pod or container. The exception is when the files are copied to a mounted persistent volume path. Do not copy files to a pod that you intend to persist, if the target path is not a peristent volume mount.

If the files being copied to a pod are intended to be permanent and a part of container image by default, the container's base image should be rebuilt to include the files.




