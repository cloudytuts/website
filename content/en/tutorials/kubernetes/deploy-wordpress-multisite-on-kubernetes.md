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

## WordPress Multisite Deployment Manifest
Finally, with our configuration information stored in `secrets` and `configMaps`, and a `persistentVolumeClaim` defined for our storage requirements, we can now create our `deployment` manifest for WordPress.

A deployment manifest will ensure there are always at least 1 replicas of a pod running. If a pod fails a new one will be scheduled to replace it.

Create a new file named `wordpress-deployment.yaml` and add the following contents.

```yaml
apiVersion: app/v1
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
      containers:
      - name: wordpress
        image: wordpress:5.5.0-php7.2-apache
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

Create your deployment by applying the manifest above to your cluster.

```shell
kubectl apply -f wordpress-deployment.yaml
```



