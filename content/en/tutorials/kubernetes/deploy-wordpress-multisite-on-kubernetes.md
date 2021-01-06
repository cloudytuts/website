---
title: "Deploy Wordpress Multisite on Kubernetes"
date: 2020-08-23T00:10:48-04:00
draft: false
author: serainville
tags:
  - kubernetes
  - wordpress
description: |
    Learn how to deploy WordPress with multisite enabled on Kubernetes with persistent storage, secrets for senstivie values, and configMaps.
---

In this tutorial you will learn how to deploy WordPress on Kubernetes with a multisite configuration. WordPress Multisite allows blog owners to host multiple sites from a single installation. This allows you manage all sites through a single pane of glass, minimizing your infrastructure spend and operational costs.

## WordPress Configurations
We're going to use the official WordPress Docker image in this tutorial. WordPress offers images based on Apache and FPM, but we will be using the Apache image as a demonstration.

WordPress Docker images are configurable by setting environment variables. These environment variables must be set at container run-time to be available to WordPress.

### Database Configuration
The primary configuration of a WordPress installation:
* `WORDPRESS_DB_HOST`
* `WORDPRESS_DB_USER`
* `WORDPRESS_DB_PASSWORD`

An optional, but strongly recommended, configuration is the table prefix.
* `WORDPRESS_DB_PREFIX`

### Multisite Configuration
Multisite does not have a dedicated environment variable. Instead, we will need to enable it via the `WORDPRESS_CONFIG_EXTRA` environment variable.

```shell
WORDPRESS_CONFIG_EXTRA="define('WP_ALLOW_MULTISITE'), true);"
```

## Storing Configs in Kubernetes
### Secret Manifest
The database username and password should not be stored in clear text. Kubernetes has a secret resource that is specifically for storing sensitive information.

{{< note >}}
Notice that we are not creating a YAML file hosting our secrets. It is strongly recommended you DO NOT store secret manifests in version control. In this tutorial, we are creating the secret from command-line to prevent a manifest from accidently being added to our Git repository.
{{< /note >}}

```shell
kubectl create secret generic wordpress-secret \
--from-literal=wordpress.db.user="wordpress_user" \
--from-literal=wordpress.db.password='my-super-secret-password'
```



### ConfigMap Manifest
Information that is not sensitive can be stored in a configMap. Let's create a configMap to hold our database name, database prefix, and extra configurations.

Create a new manifest file named `wordpress-configmap.yaml` and add the following contents:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: wordpress-configmap
data:
  wordpress.db.host: mysql.host:3306
  wordpress.db.name: wordpress_site
  wordpress.db.prefix: blog_
  wordpress.config.extra: |
    define('WP_ALLOW_MULTISITE', true);
```

To create the ConfigMap in your Kubernetes cluster use the `kubectl apply` command.

```shell
kubectl apply -f wordpress-configmap.yaml
```

## Persistent Storage
Containers are ephemeral by design. This is problematic for long lived applications like WordPress, who's state changes frequently with uploaded content, updated themes, and updated plugins. To ensure our state persists beyond the life of the container running our site, we will need to create and mount a storage volume.

Create a PersistentVolumeClaim manifest file named `wordpress-pvc.yaml` and add the following contents.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

Create the PersistentVolumeClaim in your cluster by applying the manifest file.
```shell
kubectl apply -f wordpress-pvc.yaml
```

## Service Account
Every Kubernetes Pod has an identity assigned to it. By default, if not service account is specified, a pod will be assigned the `default` service account as its identity. As a best practice, every service you run in Kubernetes should be assigned it own service account. 

Create a new service account for your WordPress pods using the following as an example.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: wordpress-multisite-sa
automountServiceAccountToken: false
```

### Tokens
Every account in Kubernetes is assigned a unique token, which used by the API server to authenticate RESTful API requests. Tokens are stored as Secrets in the namespace a Service Account is created in, and it is mounted as file of every pod that uses the service account. 

If a pod has no need to make API requests directly to Kubernetes, which is the case for nearly all workloads, the token should not be automatically mounted. In the example ServiceAccount manifest above, we've set `automountServiceAccountToken` to `false`, so that every deployment or pod that uses the `wordpress-multisite` service account does not mount the token automatically.


## WordPress Multisite Deployment Manifest
Finally, with our configuration information stored in `secrets` and `configMaps`, and a `persistentVolumeClaim` defined for our storage requirements, we can now create our `deployment` manifest for WordPress.

A deployment manifest will ensure there are always at least 1 replicas of a pod running. If a pod fails a new one will be scheduled to replace it.

Create a new file named `wordpress-deployment.yaml` and add the following contents.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-multisite
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress
  template:
    metadata:
      labels:
        app: wordpress
    spec:
      serviceAccountName: wordpress-multisite-sa
      containers:
      - name: wordpress
        image: wordpress:5.6.0-php7.3-apache
        securityContext:
          capabilities:
            drop: ["ALL"]
            add: ["NET_BIND_SERVICE", "CHOWN"]
        ports:
        - containerPort: 80
        env:
        - name: WORDPRESS_DB_HOST
          valueFrom:
            configMapKeyRef:
              name: wordpress-configmap
              key: wordpress.db.host
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: wordpress-secret
              key: wordpress.db.user
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: wordpress-secret
              key: wordpress.db.password
        - name: WORDPRESS_DB_NAME
          valueFrom:
            configMapKeyRef:
              name: wordpress-configmap
              key: wordpress.db.name
        - name: WORDPRESS_DB_PREFIX
          valueFrom:
            configMapKeyRef:
              name: wordpress-configmap
              key: wordpress.db.prefix
        - name: WORDPRESS_CONFIG_EXTRA
          valueFrom:
            configMapKeyRef:
              name: wordpress-configmap
              key: wordpress.config.extra
      volumes:
      - name: wordpress-persistent-storage
        persistentVolumeClaim:
          claimName: wp-pv-claim
```

### SecurityContext
While Docker and Containerd apply some sane defaults to protect containerized workloads, OCI containers are still largely vulnerabily and place your host and other workloads at risk. Even more can be and should be done to harden your processes to protect against malicious users.

Malicious users who find there way into your container via a flaw in your application will quickly realize they are in a container. Knowing this, they will attempt to ***escape*** the container in order to attack the host and all other containerized workloads. The key to container escapes is root privileges within the container and, unfortunately, the official WordPress images must run as root. While not good, it's not completely terrible. 

One way to protect a container and mitigate risks of escapes is to remove all Kernel capabilities granted to the parent processes' UID (0, root), except those absolutely required to run our app.

The deployment manifest above includes the following `securityContext` for our Wordpress container. With it we instruct Kubernetes to drop all Linux kernel capabilities granted to the parent process (1) who runs as root (0), and then add only `NET_BIND_SERVICE` and `CHOWN`.

* The `NET_BIND_SERVICE` capability is required in order for the web server to bind to a privileged port (1-1024). In the case of a typical web application, that's port 80.
* The `CHOWN` capability is required as part of the Wordpress container's startup process, where it applies new ownership to all Wordpress files.

```yaml
        securityContext:
          capabilities:
            drop: ["ALL"]
            add: ["NET_BIND_SERVICE", "CHOWN"]
```

By dropping all other Linux kernel capabilities we effectively neuter the root user's privileges within the container. Since all processes are limited to the kernel capabilities granted to the parent process (PID 1), even new processes are limited to the capabilities of the parent. Therefore, a malicious user would not be able to create a new process with elevated kernel capabilities permissions within the container.


Create your deployment by applying the manifest above to your cluster.

```shell
kubectl apply -f wordpress-deployment.yaml
```



