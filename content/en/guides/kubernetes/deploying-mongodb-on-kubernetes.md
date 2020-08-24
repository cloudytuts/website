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

MongoDB is an opensource NOSQL document database server popular with Node projects. 

## Objectives
* Learn how to store MongoDB credentials securely with Kubernetes Secrets.
* Learn how to store MongoDB configurations using Kubernetes ConfigMaps.
* Learn how to use Kubernetes Deployments for high availability.
* Learn how to persist data using Persistent Volume Claims.
* Learn how to backup your MongoDB databases using Kubernetes CronJobs.

## MongoDB Docker Image
Unfortunately, there is no official Docker image for MongoDB available. A Docker Community image, however, is available. As with any non-official image, it is recommended you scan the images for vulnerabilities and analyze its Dockerfile.

Dockerfiles and build scripts can be found in the [MongoDB Docker Community image](https://github.com/docker-library/mongo) repository.


## MongoDB Configurations

The Docker Community MongoDB image provides a few environment variables used to configure the database server. 

* `MONGO_INITDB_ROOT_USERNAME`
* `MONGO_INITDB_ROOT_PASSWORD`
* `MONGO_INITDB_DATABASE`

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

Create a new file named `mongodb-secrets.yaml` and add the following contents. All data values under the `data` key must be base64 encoded. Values stored under the optional `stringData` key do not require being base64 encoded.

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
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
    volumes:
    - name: mysql-persistent-storage
      persistentVolumeClaim:
        claimName: mysql-pv-claim
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
apiVersion: v1
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
            image: mongodb:5.1.1
            command:
            - "/bin/mongodump"
            - gsutil cp mongo-backup.json gs://my-project/backups
            volumeMounts:
            - name: mongodb-database-volume
              mountPath: /where/ever
        restartPolicy: OnFailure
        volumes:
        - name: mongodb-database-volume
          persistentVolumeClaim:
            claimName: application-code-pv-claim
```

## Administering MongoDB in Kubernetes
While a MongoDB likely shouldn't be exposed outside of the Kuberentes cluster, an operator is still able to make a network connection with it. 

The `kubectl port-forward` command allows us to create a proxied connection for your client machine to the Kubernetes service. For MongoDB this means we are able to make a mongodb client connection to our server.

```shell
kubectl port-forward mongodb-service 27017 &
```

With the proxy in place you can point the mongodb client to the server instance from your localhost. 

```shell
mongo --hhost localhost --port 27017 --username $MONGODB_USERNAME --password $MONGODB_PASSWORD
```
