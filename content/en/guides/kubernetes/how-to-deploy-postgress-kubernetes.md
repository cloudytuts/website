---
title: "How to Deploy Postgress Kubernetes"
date: 2020-08-25T11:59:28-04:00
draft: false
author: serainville
tags:
  - postgres
  - kubernetes
description: |
    Learn how to deploy a PostgreSQL container instance in Kubernetes with persistent storage, configMaps, and secrets. Also, how to backup a PostgreSQL database in Kubernetes.
repo: https://github.com/cloudytuts/kubernetes-in-action
---

In this guide, you will learn how to deploy and run a Postres database server instance in Kubernetes. 

## What's Covered
* Deploying containerized Postgress instances in Kubernetes
* Using Kubernetes Secrets to store sensitive configurations
* Using Kubernetes ConfigMaps to store non-sensitive configurations
* Using Kuberentes Persistent Volumes to persist databases
* Using Kubernetes CronJobs to backup databases


## PostgreSQL Docker Image
PostgreSQL offers an [Official PostgreSQL Docker image](https://hub.docker.com/_/postgres) through Docker Hub. It always recommended to use official images over third-party or self-created images. In this guide we will use the official image to deploy the database server in Kubernetes.

### Environment Variables
Environment variables allow us to configure environment specific parameters for Docker images that support them. The official Postgres images support the following environment variables.

* **POSTGRES_PASSWORD** *(required)* is used to set the superuser password.
* **POSTGRES_USER** *(optional)* is used to create a superuser with the name set in the environment variable.
* **POSTGRES_DB** *(optional)* is used to create a database to be used as the default.

We will dive into setting these values in the next section: Configuring PostreSQL. For now, it's good to understand what variables are available for configuring your instance.

## Configuring PostgreSQL
### ConfigMaps
ConfigMaps provide a means to store environment parameters in Kubernetes, to be fetched by a Pod when it starts. Values in a ConfigMap and be key-pair strings, entire files, or both. Which you use depends on your implementation.

Data stored in a ConfigMap not encrypted. These Kubernetes resources should be only be used with data that is non-sensitive.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: postgres
data:
  POSTGRES_DB: myapp_production
```

### Secrets
For sensitive data, such as user credentials, Kubernetes Secrets allow you to more safely store data in the cluster. Like ConfigMap, the values in a Secret can be fetched by a Pod during startup.

Kubernetes Secrets can be expressed using a manifest, like any other resource. However, due to their sensitive nature it is not recommended to store secret manifest files in your version control system with actual values. 

To create a secret resource named `postgres` for storing a superuser username and password, use the `kubectl create secret` command.

```shell
kubectl create secret generic postgres \
--from-literal=POSTGRES_USER="root" \
--from-literal=POSTGRES_PASSWORD="my-super-secret-password"
```

Alternatively, if you do wish to use a manifest file, create a new `postgres-secrets.yaml` file with the following structure.

{{< note >}}
If you choose to store your secret manifest in version control, consider removing the actual values. By keeping just the structure your operators will know which parameters must be set.
{{< /note >}}

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: postgres
data:
  POSTGRES_PASSWORD: bXktc3VwZXItc2VjcmV0LXBhc3N3b3Jk
stringData:
  POSTGRES_USER: root
```

Notice the value used for `POSTGRES_PASSWORD`. That value is not the actual password. Rather, it is base64 encoded string of the password. Do not confuse base64 encoding with encyption. It merely serves to obfuscate the password to prevent prying eyes from easily reading it. 

To base64 encode a string in Linux and OSX, you use the `base64` command.

```shell
echo -n 'my-super-secret-password' | base64
```

To create the secret resource for Postgres, apply the manifest.

```shell
kubectl apply -f postgres-secrets.yaml
```

## Deploying PostgreSQL
While containers in Kubernetes run from Pods a Postgres server should not be deployed as a pod. Rather, a Kubernetes Deployment should be used. 

### Persistent Volume Claim
Database servers are stateful and, therefore, their databases are expected to be persistent. A container, on the other hand, is ephermeral with no expectation for persisting state. This is obviously problemmatic with stateful applications like database servers.

A Kubernetes Persistent Volume is a volume that attaches to a Pod or group of Pods. Data writtent to it persists and is available to any Pod that mounts it. In most use cases it can be thought of as a network filesystem (NFS), as the behaviour is similar.

To mount a Persistent Volume to a Pod a Persistent Volume Claim must exist.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pv-claim
  labels:
    app: postgres
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

### Deployment Manifest

A Kubernetes deployment manifest for Postgres would look similar to the example. 
* Create a single pod replica.
* Use a base image Postgres 12.4 runing on Alpine Linux.
* Expose port 5432.
* Environment variables set using parameters found in a Secret manifest
* Environment variables set using parameters found in a ConfigMap.
* Mount a Persistent Volume using a Persistent Volume Claim for the Postgres database files.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres  
    spec:
      containers:
      - name: postgres
        image: postgres:12.4-alpine
        ports:
          - containerPort: 5432
        envFrom:
          - secretRef:
              name: postgres-secrets
          - configMapRef:
              name: postgres-configmap  
        volumeMounts:
        - name: postgres-database-storage
          mountPath: /var/lib/pgsql/data
      volumes:
      - name: postgres-database-storage
        persistentVolumeClaim:
          claimName: postgres-pv-claim
```



## Exposing PostgreSQL
Services running in Kubernetes are exposed using Kubernetes Services. The following Service manifest will attach to our Postgres deployment and accept traffic over TCP port 5432. 

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  selector:
    app: postgress
  ports:
  - protocol: TCP
    port: 5432
  clusterIP: none
```

In the manifest above we've chosen not to provide an internal Cluster IP address to our Postgres service. We do not want the service to be exposed outside of the cluster. Rather, we want to force applications to connect to this service using Kubernetes' CoreDNS.

The DNS name for this service will be the following.
```text
postres.default.svc
```
The `default` part of the name is derived from the resource's namespace. Since we have not set a namespace for our Postgre reources, they will automaticaly default to the default namespace.


## Backing up Databases
As with any database service, backups should be done routinely in order to safe guard your data. While you could `kubectl exec` into the running Postgres Pod and run the `pg_dump`, that's a manual process that should be avoided. 

Instead, we are going to create a CronJob in order for us to automate the task. A Kuberentes CronJob is a scheduled job that schedules a pod for executing commands. Our CronJob will perform the following tasks.

* Use the Google Cloud SDK Docker image
* Mount the Postgre database persistent volume.
* Use the Postgres ConfigMap and Secret to inject environment variables
* Execute `pg_dump` against the database
* Copy the database dump to a Google Cloud Storage Bucket.

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: postgres-backup
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: postgres-backup
            image: google/cloud-sdk:alpine
            args:
            - apk --update add postgresql
            - pg_dump -u myblog > myblog-$(date +%s).bak
            - gsutil cp myblog.bak gs://myblog/backups
            envFrom:
            - secretRef:
                name: postres-secrets
            - configMapRef:
                name: postgres-configmap
            volumeMounts:
            - name: postgres-database-storage
              mountPath: /var/lib/pgsql/data
          volumes:
          - name: postgres-database-storage
            persistentVolumeClaim:
              claimName: postgres-pv-claim         
```

