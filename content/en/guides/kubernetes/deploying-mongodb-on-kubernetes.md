---
title: "Deploying Mongodb on Kubernetes"
date: 2020-08-23T08:53:41-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - mongodb
description: |
    Learn how to deploy and run highly available MongoDB Kubernetes services by utilizing deployments, secrets, configMaps, and persistent volumes. 
repo: https://github.com/cloudytuts/kubernetes-in-action/tree/master/mongodb/basic-example
---

MongoDB is an opensource, general purpose, document-based, distributed NOSQL database server that is especially popular with JavaScript projects. In this guide you will learn how to deploy and run MongoDB as a container in Kubernetes. 


## Objectives
* Learn how to store MongoDB credentials securely with Kubernetes Secrets.
* Learn how to store MongoDB configurations using Kubernetes ConfigMaps.
* Learn how to use Kubernetes Deployments for high availability.
* Learn how to persist data using Persistent Volume Claims.
* Learn how to backup your MongoDB databases using Kubernetes CronJobs.

## MongoDB Docker Image
Unfortunately, there is no official Docker image for MongoDB available. A Docker Community image, however, is available. As with any non-official image, it is recommended you scan the images for vulnerabilities and analyze its Dockerfile.

Dockerfiles and build scripts can be found in the [MongoDB Docker Community image](https://github.com/docker-library/mongo) repository.


## MongoDB Docker Configurations

The Docker Community MongoDB image provides a few environment variables used to configure the database server. 

* `MONGO_INITDB_ROOT_USERNAME`
* `MONGO_INITDB_ROOT_PASSWORD`
* `MONGO_INITDB_DATABASE`

The first two are used to set the root username and password for administering a MongoDB server. With MongoDB in general and the Docker Community image, these two values should be set. 

{{< warning >}}
If `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD` are not set Mongo will default to passwordless authentication. Anyone who connects to your instance will have full root access.
{{< /warning >}}

### Secrets
Secrets provide a means to safely store sensitive information in Kubernetes. Data stored as a secret is base64 encoded, to obscure the values when displayed on screen, and is stored encrypted on the ETCD database server used by Kubernetes. 

The two configuration values that standout as requiring extra protection are `MONGO_INITDB_ROOT_USERNAME` and `MONGO_INITDB_ROOT_PASSWORD`. 

To create a Kubernetes secret for the MongoDB root user and password you could run the following `kubectl create` command.

```shell
kubectl create secret generic mongodb \
--from-literal="root" \
--from-literal='my-super-secret-password'
```

Alternatively, a Secret manifest can be written and applied to create the MongoDB secrets.
{{< warning >}}
Secret manifests should not be stored in version control systems, especially in the same repository as your other manifests.
{{< /warning >}}

Kubernetes stores secrets as base64 encoded strings. Secret manifests give you the option of writing the secret value as a regular string or a base64 encoded string. To prevent exposure from someone looking over your shoulder it is advisable to always base64 encode passwords.

To base64 encode a password string use the `base64` command available on Linux and OSX.

```shell
echo -n 'my-super-secret-password' | base64
```
    | Output
```shell
bXktc3VwZXItc2VjcmV0LXBhc3N3b3Jk
```

Create a new file named `mongodb-secrets.yaml` and add the following contents. Secrets can be stored as `data` or `stringData` in the Secret manifest. All data values under the `data` key must be base64 encoded. Values stored under the optional `stringData` key do not require being base64 encoded.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb-secret
data:
  MONGO_INITDB_ROOT_PASSWORD: bXktc3VwZXItc2VjcmV0LXBhc3N3b3Jk
stringData:
  MONGO_INITDB_ROOT_USERNAME: myroot
```

Apply to manifest to create the resource in your Kubernetes cluster.

```shell
kubectl apply -f mongodb-secrets.yaml
```

### MongoDB Deployment Manifest
A deployment describes a desired state to be enforced by the Deployment Controller. When the state of a deployment shifts away from the desired state, the Deployment Controller will perform actions to bring the state back. 

One example of maintaining state is ensuring a set number of replica Pods is running. If a Deployment Pod fails the Deployment Controller will replace the failed Pod. 

Updates and Rollbacks: When a Pod template is updated in a Deployment resource, the Deployment Controller will deploy updated Pods before removing the older Pods. The new pods will not replace the older ones unless the start with a health state.

Create a new file named `mongodb-deployment.yaml` and add the following contents.

```yaml
apiVersion: app/v1
kind: Deployment
metadata:
  name: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongodb:3.6.19-xenial
        ports:
          containerPort: 27017
```

The example above will deploy a single replica of a MongoDB instance. The base image will use the Docker Community MongoDB image at version 3.6.19, which is based on Ubuntu Xenial.

Apply the manifest to create the deployment resource in Kubernetes.
```shell
kubectl apply -f mongodb-deployment.yaml
```

### Custom MongoDB Config File
Our MongoDB pod uses the default configuration for a newly installed server. The only two items configured thus far are the username and password.

There a are plenty of other configurations options in a MongoDB server. However, these options are configured from within a `mongodb.conf` file. 

### Create ConfigMap
To use a `mongodb.conf` file with your MongoDB instance on Kubernetes you must create a file and store it as a configMap item. 

The following is an example of a `mongodb.conf` file. 

```yaml
systemLog:
   destination: file
   path: "/var/log/mongodb/mongod.log"
   logAppend: true
storage:
   journal:
      enabled: true
processManagement:
   fork: true
net:
   bindIp: 127.0.0.1
   port: 27017
setParameter:
   enableLocalhostAuthBypass: false
```

To create a `configMap` item of the `mongodb.conf` file, run the following `kubectl` command. The following example create a **ConfigMap** named **mongodb**, if it doesn't already exits, and adds the `mongodb.conf` file to a item named `conf`.

```shell
kubectl create configMap mongodb-config-file --from-file=conf=mongodb.conf
```

### Mount mongodb.conf file
The simplist way of injecting the configuration file into your MongoDB Pod is by mounting the file as a volume. The configuration file will then be avaibable as file to the MongoDB service when its container starts.

To mount the configuration file as a volume we need to update our Deployment manifest.

```yaml {hl_lines=["20-27"]}
apiVersion: app/v1
kind: Deployment
metadata:
  name: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongodb:3.6.19-xenial
        ports:
          containerPort: 27017
        volumeMounts:
        - name: mongodb-configuration-file
          mountPath: /etc/mongod.conf
          readOnly: true
    volumes:
    - name: mongodb-configuration-file
      configMap:
        name: mongodb-config-file
```


## Persistent Storage
Containers are ephemeral by design. Any state changes to the container will be lost when the container stops. For a database server like MongoDB, this means your entire database will be wiped. 

Persistent Volumes can be mounted to a Pod allowing data to persist beyond the life of a container. To add persistent volumes to a container in Kubernetes you will need to create a **Persistent Volume Claim** and then mount the volume to a **Deployment**.

Create a file named `mongodb-pvc.yaml` and add the following contents.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mongodb-pv-claim
  labels:
    app: mongodb
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

The volume claim will createa writable storage volume, provisioned by your cloud provider, with a 1GB size. 

Apply it to the cluster using the `kubectl apply` command.
```shell
kubectl apply -f mongodb-pvc.yaml
```

To mount the volume we need to update our deployment manifest. A `volume` needs to be added to the `template` key in our manifest, and a `volumeMount` with the `container` key to mount the `volume`. We've highlighted both below.

```yaml {hl_lines=["24-25","30-32"]}
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongodb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      containers:
      - name: mongodb
        image: mongo:4.4.0-bionic
        ports:
          - containerPort: 27017
        envFrom:
        - secretRef:
            name: mongodb-secret
        volumeMounts:
        - name: mongodb-configuration-file
          mountPath: /etc/mongod.conf
          readOnly: true
        - name: mongodb-persistent-storage
          mountPath: /data/db
      volumes:
      - name: mongodb-persistent-storage
        persistentVolumeClaim:
          claimName: mongodb-pv-claim
      - name: mongodb-configuration-file
        configMap:
          name: mongodb-config-file
```

Apply your deployment manifest to update an existing deployment or to create one.
```yaml
kubectl apply -f mongodb-deployment.yaml
```

## Exposing Service
So far we've deployed a single Pod of a MongoDB instance. While this single pod can be exposed as a service it is strongly discouraged, outside of testing and development. Pods are ephemeral and once they stop their state is lost, which includes the assigned IP address. 

### Internal Service
An internal service is a service the is only accessible from within the Kubernetes cluster. This is the default behavoir of a service. For database server and other backend systems this is the most likely service configuration.

Create a new service for the MongoDB instance.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  selector:
    app: mongodb
  ports:
  - protocol: TCP
    port: 27017
```
  
Apply the manifest to the Kubernetes cluster to create the service resource.
```shell
kubectl apply -f mongodb-service.yaml
```

## Backing Up MongoDB
### CronJob
A CronJob is a scheduled, containerized job. 

A CronJob backup for MongoDB will perform the following:
* Run MongoDB container image
* Mount volume used by MongoDB instance
* Execute `mongodump` command to dump database
* Copy dump to a storage bucket, such as a Google Cloud Storage Bucket.

```yaml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: mongodb-backup
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: mongodb-backup
            image: mongo:4.4.0-bionic
            args:
            - "/bin/sh"
            - "-c"
            - "/usr/bin/mongodump -u $MONGO_INITDB_ROOT_USERNAME -p $MONGO_INITDB_ROOT_PASSWORD -o /tmp/backup -h mongodb"
            - "tar cvzf mongodb-backup.tar.gz /tmp/backup"
            #- gsutil cp mongodb-backup.tar.gz gs://my-project/backups/mongodb-backup.tar.gz
            envFrom:
            - secretRef:
                name: mongodb-secret
            volumeMounts:
            - name: mongodb-persistent-storage
              mountPath: /data/db
          restartPolicy: OnFailure
          volumes:
          - name: mongodb-persistent-storage
            persistentVolumeClaim:
              claimName: mongodb-pv-claim
```

## Administering MongoDB in Kubernetes
While a MongoDB likely shouldn't be exposed outside of the Kuberentes cluster, an operator is still able to make a network connection with it. 

### Port Forward
The `kubectl port-forward` command allows us to create a proxied connection for your client machine to the Kubernetes service. For MongoDB this means we are able to make a mongodb client connection to our server.

```shell
kubectl port-forward mongodb-service 27017 &
```
  | Output
```shell
Forwarding from 127.0.0.1:27017 -> 27017
Forwarding from [::1]:27017 -> 27017
```

With the port forwarded from our local machine to the MongoDB service we can use the `mongo` client to conenct.

```shell
mongo -u <root-username> -p <root-password>
```
  | Output
```shell
MongoDB shell version v4.2.0
connecting to: mongodb://127.0.0.1:27017/?compressors=disabled&gssapiServiceName=mongodb
Handling connection for 27017
Implicit session: session { "id" : UUID("feb9a859-43eb-4cb3-bdd9-ef76690cbb92") }
MongoDB server version: 3.6.19
WARNING: shell and server versions do not match
Server has startup warnings: 
2020-08-24T03:21:23.861+0000 I STORAGE  [initandlisten] 
2020-08-24T03:21:23.861+0000 I STORAGE  [initandlisten] ** WARNING: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine
2020-08-24T03:21:23.861+0000 I STORAGE  [initandlisten] **          See http://dochub.mongodb.org/core/prodnotes-filesystem
> 
```

### Interactive Container Shell
Alternatively, an interactive shell can be opened into the running MongoDB container. You will need the name of your MongoDB pod in order to shell into it.

```shell
kubectl get pods
```
  | Output
```shell
NAME                       READY   STATUS    RESTARTS   AGE
mongodb-7cfbc6f555-t97c4   1/1     Running   0          12m
```

We are only running a single replica and its name is `mongodb-7cfbc6f555-t97c4`. Now that we have the Pod's name we can shell into it.

```shell
kubectl exec -it mongodb-7cfbc6f555-t97c4 /bin/bash
```
  | Output
```shell
root@mongodb-7cfbc6f555-t97c4:/# 
```

When the interactive shell for the container created we can manage MongoDB using Mongo's interactive shell.

```shell
mongo -u $MONGO_INITDB_ROOT_USERNAME -p $MONGO_INITDB_ROOT_PASSWORD
```


