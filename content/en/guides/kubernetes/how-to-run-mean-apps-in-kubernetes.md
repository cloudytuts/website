---
title: "How to deploy MEAN apps in Kubernetes"
date: 2020-08-24T20:58:15-04:00
draft: false
author: serainville
tags:
  - mean
  - mongodb
  - kubernetes
  - express
  - angular
description: |
    Learn how to deploy and run a MERN app stack in Kubernetes
repo: https://github.com/cloudytuts/kubernetes-in-action
---

In this guide, you will learn how to deploy and run a MERN stack in Kubernetes. This guide will walk you through the entire stack of MERN application. You will learn how to deploy a MongoDB server, your Express.js app, and finally your Angular app.

Each part of the stack needs to configured. You will learn how to use `configMaps` and `secrets` to store your configurations.

## What You Will Learn
* How to deploy and run a MongoDB server in Kubernetes.
* How to deploy and run an Express.js backend app in Kubernetes.
* How to deploy and run a Angular app in Kubernetes.
* How to use configMaps and Secrets to store environmental configurations for your services.
* Persistent Volumes for persisting MongoDB databases.

## Dockerize React App
Angular apps transpile into static web files during their build process. Kubernetes does not support service static web files natively, so instead we will need to create a Docker image for our Angular app. 

A web server will be need to serve our Angular app. The most common static file web server is NGINX, made popular by its resource efficincies and performance. We will create an NGINX-based Docker image for the Angular app.

### Dockerfile
Our Dockerfile will require two stages to ensure we output the slimest and most secure image possible. Angular apps, and JavaScript apps based on Node in general, use large modules directory to store the source files of its dependancies. It also requires a few build tools for compiling and testing your app.

Build tools and the node_modules directory should not be included in your final image. Our first stage will handle building and testing our application.

Our second stage will copy our app's build artifacts into an NGINX-based image.

Create a new file named `Dockerfile` in your project directory. Add the following contents to it.

```dockerfile
# Build and test stage
FROM node:14.8.0-stretch AS build
COPY ./src .
RUN npm install \
    && npm run-script test \
    ** npm run-script build

# Final stage
FROM nginx:1.19.2-alpine AS final
COPY --from=build build/ /usr/local/nginx/html
```

To build the Docker image for your Angular app, run the `docker build` command and name your image using the `-t` flag.
```shell
docker build -t angular-app:1.0.0 .
```

If your Kubernetes is remotely hosted, such GKE, AKE, or Digital Ocean Kubernetes, you will need to upload your image to a repository accessible to your cluster.

Tag your newly built image for Docker Hub, for example, add a new tag for your image with the Docker Hub project name.
```shell
docker tag angular-app:1.0.0 myproject/angular-app:1.0.0
```

Push your newly tagged image to Docker Hub. 
```shell
docker push myproject/angular-app:1.0.0
```

## Dockerize Express App
Follwing the same strategy as the Angular app, we will be creating a multistage Docker build for the Express app. The major difference between them is the final image. Our final image will be based on Node and it will install pm2 to run our backend Express service.

```dockerfile
# Install build dependancies and build Express app
FROM node:14.8.0-stretch AS build
COPY ./src .
RUN npm install \
    && npm run-script test \
    ** npm run-script build

# Create final image and use PM2 to manage Express app
FROM node:14.8.0-slim AS final
RUN mkdir /app
WORKDIR /app
COPY build/ .
RUN npm install -g pm2
CMD ["pm2", "start", "server.js"]
```

To build the Docker image for your Express app, run the `docker build` command and name your image using the `-t` flag.
```shell
docker build -t express-app:1.0.0 .
```

If your Kubernetes is remotely hosted, such GKE, AKE, or Digital Ocean Kubernetes, you will need to upload your image to a repository accessible to your cluster.

Tag your newly built image for Docker Hub, for example, add a new tag for your image with the Docker Hub project name.
```shell
docker tag express-app:1.0.0 myproject/express-app:1.0.0
```

Push your newly tagged image to Docker Hub. 
```shell
docker push myproject/express-app:1.0.0
```


## Deploy Mongodb
### Configurations
By default, a MongoDB instance will allow any unauthenticated user to access it and browse its databases. In a production environment, this is undesirable for obvious reasons. The Docker Community managed MongoDB image supports two environment variables for setting a root user and root password. 

Credential should never be stored in clear text, which makes a Kubernetes `secret` the best resource type for storing our root credentials.

The following command will create a secret named `mongodb` and store a root username and root password key.

```shell
kubectl create secret generic mongodb \
--from-literal=MONGO_INITDB_ROOT_USERNAME="root" \
--from-literal=MONGO_INITDB_ROOT_PASSWORD='my-super-secret-password'
```

Alternatively, a secret can be defined using a YAML manifest file. Due to the sensitive nature of secrets, a Secrets manifest should not be stored in a version control system along with your other manifests. However, for demonstration purposes an example can be seen below.

If you decide to use a manifest file to define your MongoDB secrets, create a new file named `mongodb-secrets.yaml` and add the following.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mongodb
data:
  MONGO_INITDB_ROOT_PASSWORD: c3VwZXItc2VjcmV0LXBhc3N3b3Jk
stringData:
  MONGO_INITDB_ROOT_USERNAME: root
```

The `MONO_INITDB_ROOT_PASSWORD` value is a base64 encoded string of the root pasword. Do not confuse this with being encrpyted. A base64 encoded string merely serves as a way to protect your values from those looking over your shoulder. Any value for a key under the `data` key must be base64 encoded.

To base64 encode a string on Linux or OSX, use the `base64` command.
```shell
echo -n "my-super-secret-string" | base64
```

The `MONGO_INITDB_ROOT_USERNAME` is not as sensitive as the password. The example above show its value as a regular, non-encoded string. Any value for a key under `stringData` may be added non-encoded.

Use the `kubectl apply` command to create the `mongodb` secret on your Kubernetes cluster.
```shell
kubectl apply -f mongodb-secrets.yaml
```

### Persistent Volume
A database server's databases should persist beyond the life of a dabase server instance. For this reason it is strongly recommended to use a Persistent Volume to store your database(s). Kubernetes supports Persistent Volumes through Persistent Volume Claims.

Create a new Persistent Volume Claim manifest named `mongodb-pvc.yaml` and add the following contents.

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

The manifest above will create a claim for a 1GB storage volume, provided by your Cloud Provider.

Use the `kubectl apply` command to create the PersistentVolumeClaim resource in your cluster.
```shell
kubectl apply -f mongodb-pvc.yaml
```


### Deployment
We're finally ready to deploy our MongoDB server. We'll tie in the `secret` and `PersistentVolumeClaim` resources above to configure our instance.

Create a new file named `mongodb-deployment.yaml` and add the following contents.

```yaml
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
        - name: mongodb-persistent-storage
          mountPath: /data/db
      volumes:
      - name: mongodb-persistent-storage
        persistentVolumeClaim:
          claimName: mongodb-pv-claim
```
The manifest above will do the following:
* Deploy MongoDB 4.4.0 on an Ubuntu Bionic based container image
* Mount a persistent volume for the `/data/db` directory, the default database directory for Mongo.
* Set the Root username and password using the values from the `secret` manifest.
* Create a single replica of the Mongo instance.

To create the deployment resource on your Kubernetes cluster use the `kubectl apply` command.
```shell
kubectl apply -f mongodb-deployment.yaml
```

### Service
Lastly, your applications should not connect with the MongoDB pod directly. A Pod is ephemeral, meaning it could disappear at anytime. While the deployment will automatically reschedule a new pod to take the place of the failed one, its name and IP address will be different.

Instead, a Service resource should be used. We will create a service resource that is only accessible from with your Kubernetes cluster. As this is a backend service for your application, there is no need for any other applications to access it.

Create a new file named `mongodb-service.yaml` and add the following contents.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb
  labels:
    app: mongodb
spec:
  ports:
    - port: 3306
  selector:
    app: mongodb
  clusterIP: None
```

Create the new service resource by running the `kubectl apply` command.
```shell
kubectl apply -f mongodb-service.yaml
```

While we've set the `clusterIP` to `None`, the service is still accessible from within your cluster. To access it you will need to use it's hostname. A service in Kubernetes can be accessed by `<service-name>.<namespace>.svc`.  

We have not defined a namespace for our demostration, to simplify the example. This means the MongoDB service will reside in the `default` namespace. To access this Mongo service you would use the following hostname.

```text
mongodb.default.svc
```

### Administering Mongo
Remote administration of the Mongo server can still be accomplished from your workstation, despite the service not being given a ClusterIP or LoadBalancer IP.

To administer the Mongo server from your workstation, use the `kubectl port-forward` to create a connection between your machine and the remote service.

```shell
kubectl port-forward svc/mongodb 27017 &
```

Assuming you have a Mongo client installed, you can administer the server as if it were running locally on your machine.

```shell
mongo -u root -p'super-secret-password'
```


## Deploy Express App
### Configurations
### Deployment
### Service

## Deploy Angular App
### Configurations
### Deployment
### Service



