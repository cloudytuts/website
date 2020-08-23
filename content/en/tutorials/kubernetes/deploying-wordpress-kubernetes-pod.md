---
title: "Deploy a WordPress Kubernetes Pod"
date: 2020-08-22T20:30:29-04:00
draft: false
author: serainville
tags:
    - wordpress
    - kubernetes
    - pod
description: |
    Learn how to create and manage Kubernetes pods with WordPress in your Kubernetes cluster.
---

In this tutorial you will learn how to create and deploy a Kubernetes Pod for WordPress. 

With Kubernetes it is rarely advisable to create individual pods. A pod is an ephemeral construct, which means it does not maintain state as it is considered disposable. Therefore, a pod cannot be restarted once it has stopped. It can only be replaced. 

If your interest is running WordPress in production, we recommend you read our tutorial on usins Kubernetes Deployment instead, [how to run WordPress on Kubernetes]({{< relref "how-to-run-wordpress-on-kubernetes" >}}).

## What is a Pod
A pod is the lowest construct available in Kubernetes. While Kubernetes is a container platform a Pod is not a container itself. Rather, it is a grouping of one or more containers with one being the primary process. Secondary containers are referenced as *sidecars*. 

Pods create a shared namespace for all containers within it. There is no isolation between the containers inside, so they are able to interact with each other directly. For example, each container will reside in the same network namespace and filesystem. This allows for monitoring agents, for example, to be deployed in separate containers within a pod to pull metrics from an application running in the pod.


## Pod Manifest
A Pod Manifest is a YAML file that describes the desired state of a Kubernetes Pod. The manifest is used to define 

* The pod's name
* Container image
* Environment Variables
* Mounted Storage Volumes
* Network Ports

## WordPress Docker Image
WordPress provides an [Official Docker image](https://hub.docker.com/_/wordpress). Rather than attempting to create and maintain your own images, you with their images as a base. WordPress offers numerous base images for different use cases. The image tags use the following convention 
```text
[wordpress-version]-php[version]-[web-server]
```
We recommend using an Apache or FPM based image, such as the following two examples.

* `5.5.0-php7.2-apache`
* `5.5.0-php7.2-fpm`

{{< note >}}
The tags above are used as an example. Check the [Official WordPress Docker Hub](https://hub.docker.com/_/wordpress) for more recent images.
{{< /note >}}

### Configuring a WordPress Container
There are a number of configuration options that must be set for WordPress to run. One such configuration is the backend database. The official WordPress docker images accept a number of **environment variables** that should be set during a containers run-time. 

* `WORDPRESS_DB_HOST`
* `WORDPRESS_DB_USER`
* `WORDPRESS_DB_PASSWORD`
* `WORDPRESS_DB_NAME`
* `WORDPRESS_DB_PREFIX` (optional)[recommended]

WordPress uses SHA1 hashes to salt PHP session information for security and privacy reasons. While WordPress will automatically generate new hashes during its initial install phase, you can provide your own.

* `WORDPRESS_AUTH_KEY`
* `WORDPRESS_SECURE_AUTH_KEY`
* `WORDPRESS_LOGGED_IN_KEY`
* `WORDPRESS_NONCE_KEY`
* `WORDPRESS_AUTH_SALT`
* `WORDPRESS_SECURE_AUTH_SALT`
* `WORDPRESS_LOGGED_IN_SALT`
* `WORDPRESS_NONCE_SALT`

Extra configurations are also available. These are typically used to enable `multisite`, for example.

* `WORDPRESS_CONFIG_EXTRA`


## Managing WordPress Configurations
### Secrets
WordPress requires a database username and password. Two very senstivite values that should never be exposed to the public. To safeguard these values they should be stored as Kubernetes Secrets.

Create a new Kubernetes secret from command-line named `wordpress`. We'll use it to store our database username and password.

```shell
kubectl create secret generic wordpress-secret \
--literal-string=WORDPRESS_DB_USER=wordpress_user \
--literal-string=WORDPRESS_DB_PASSWORD='my-super-secret-password'
```

### ConfigMap
A Kubernetes configMap is used to store non-sensitive configuration values. These values are not encrypted on disk and are stored in clear text.

For WordPress we will create a ConfigMap to store our database host, table prefix, and other information that isn't senstive.

```shell
kubectl create configmap wordpress-configmap \
--from-literal=WORDPRESS_DB_HOST=mysql.host:3306
--from-literal=WORDPRESS_DB_PREFIX=blog_ \
--from-literal=WORDPRESS_DB_NAME=wordpress_blog
```

## WordPress Pod
Now that we have our WordPress configurations stored in **Secrets** and **ConfigMaps**, it's time to create our WordPress Pod  manifest file.

### Pod Manifest Example 1
The following example creates a basic Pod that runs a WordPress container. Our preconfigured secrets and configMaps are pulled in to create environment variables for WordPress. The Pod is configured to expose TCP port 80.

Create a file named `wordpress-example-1.yaml` and add the following contents.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: wordpress
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: wordpress
        image: wordpress:5.5.0-php7.2-apache
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
      envFrom:
      - secretRef:
        name: wordpress-secret
      - configRef:
        name: wordpress-configmap
        
      restartPolicy: OnFailure
```


Deploy your WordPress pod using the `kubectl` command and referencing the `wordpress-example-1.yaml` file.

```shell
kubectl apply -f wordpress-example-2.yaml
```



### Pod Manifest Example 2
The manifest above creates environment variables from secrets and configMaps. Alternatively, we can define each inside of the manifest itself, while still pulling the values from the secret and configMap resources.

Create a file named `wordpress-example-2.yaml` with the following contents.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: wordpress
spec:
  template:
    # This is the pod template
    spec:
      containers:
      - name: wordpress
        image: wordpress:5.5.0-php7.2-apache
        ports:
        - name: http
          containerPort: 80
          protocol: TCP
      env:
      - name: WORDPRESS_DB_HOST
        valueFrom:
          configMapKeyRef:
            name: wordpress-configmap
            key: WORDPRESS_DB_HOST
      - name: WORDPRESS_DB_USER
        valueFrom:
          secretKeyRef:
            name: wordpress-secret
            key: WORDPRESS_DB_USER
      - name: WORDPRESS_DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: wordpress-secret
            key: WORDPRESS_DB_PASSWORD
      - name: WORDPRESS_DB_NAME
        valueFrom:
          configMapKeyRef:
            name: wordpress-configmap
            key: WORDRESS_DB_NAME
      - name: WORDPRESS_DB_PREFIX
        valueFrom:
          configMapKeyRef:
            name: wordpress-configmap
            key: WORDPRESS_DB_PREFIX
      restartPolicy: OnFailure
```

Deploy your WordPress pod using the `kubectl` command and referencing the `wordpress-example-2.yaml` file.

```shell
kubectl apply -f wordpress-example-2.yaml
```


