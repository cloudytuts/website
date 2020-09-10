---
title: "How to Configure Angular, React, Node in Kubernetes"
date: 2020-09-09T23:07:42-04:00
draft: false
author: serainville
tags:
    - kubernetes
    - configmaps
    - secrets
    - angular
    - react
    - vue
description: |
    Learn how to configure Angular, React, and Vue apps in Kubernetes using ConfigMaps and Secrets for environment variables and config.js files.
---

In a high-velocity application development world, your application artifact should be deployable in any environment from dev to production. The only way to accomplish this is to strip your environment-specific configurations and have the application fetch it at runtime.

In this tutorial, we are going to look at different ways of storing and injecting environment variables and files into your node-based JavaScript apps, such as Angluar, React, and Vue. 



## Environment Variables
One of the easiest things you can do to store and retrieve configurations in Kubernetes is to create environment variables. Any resource in Kubernetes, from deployments and pods to CronJobs, can have environment variables injected into them.

To create environment variables in a deployment or pod, for example, you need to add the `env` key to the `containers` template. For example, the deployment example below sets three environment variables: `ENVIRONMENT`, `DB_HOST`, and `LOGGING`. 

```yaml
apiVersion: app/v1
kind: Deployment
metadata:
  name: nodejs-app
  namespace: development
spec:
  replicas: 8
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - name: nodejs-app
        image: nodejs-app:3.2.0
        ports:
          containerPort: 80
        env:
        - name: ENVIRONMENT
          name: development
        - name: DB_HOST
          value: mysql-dev
        - name: LOGGING
          value: "false"
    volumes:
    - name: nodejs-env-file
      secret:
        name: nodejs-config
```

To access an environment variable in your container you use the `name` key. For example, to access the `ENVIRONMENT` environment variable in a node app you would point to its name -- `process.env.ENVIRONMENT`. 

The most obvious downfall for storing configuration data in the deployment manifest itself is the data is not protected. Any sensitive information will be visible to those who have access to the manifest file and those who have permission on the Kubernetes cluster.

A second limitation of using this method is the environment variables cannot be shared by other pods or deployments. It's not uncommon to run multiple deployments of an app for releasing different versions, which would require each to store the environment variables separately -- risking config drifts. 


## .ENV files
An `.ENV` file is a common way of storing environment-specific parameters for node-based JavaScript applications. When the file exists in your project root and your application is using the [dotenv](https://www.npmjs.com/package/dotenv "dotenv npm page") module, parameters in the `.env` file will be injected into your app as environment variables.

The typical structure of an `.env` file looks similar to the following.
```shell
DB_HOST=mysql-prod
DB_USER=root
DB_PASS=P@ssw0rd123
```
We can store this file as either a **ConfigMap** or a **Secret** depending on its contents. When deciding which resource type to use for storing the file in Kubernetes, always place sensitive data in a **ConfigMap**. As the above example stores a `DB_USER` and `DB_PASS` parameters, it should be stored in a **Secret**.

### Storing .env as a Secret
The simplest method for creating a Kubernetes Secret and adding a file to it is to use the `kubectl create secret` command. For instance, to store the `.env` file in a secret named `react-app-prod`, you would run the following command.

```shell
kubectl create secret generic react-app-prod --from-file=.env
```
```shell
secret/react-prod-demo created
```

To verify the secret was created successfully, you can use the `kubectl describe secret` command with the name of the secret you just created.

```shell
kubectl describe secret react-prod-demo
```
```shell
Name:         react-prod-demo
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
.env:  46 bytes
```

As you can, you won't be able to see the actual data in the `.env` file stored in a Kubernetes secret, but you can verify the secret was created successfully.

### Storing .env as a ConfigMap
```shell
kubectl create configmap nodejs-env --from-file=.env
```

### Mounting .env file
With the `.env` file stored in either a ConfigMap or Secret it now needs to be mounted as a file in your Angular, Node, React, or Vue app, for example. 

```yaml
apiVersion: app/v1
kind: Deployment
metadata:
  name: nodejs-app
  namespace: production
spec:
  replicas: 8
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - name: nodejs-app
        image: nodejs-app:3.2.0
        ports:
          containerPort: 80
        volumeMounts:
        - name: nodejs-env-file
          mountPath: /app/.env
          readOnly: true
    volumes:
    - name: nodejs-env-file
      secret:
        name: nodejs-config
```

## Config.js Files


```javascript
// ------------------------------------------------------------------
// APP CONFIGURATION
// ------------------------------------------------------------------

module.exports = {
    logging: true,
    api: {
        host: `api-prod`,
        key:  `ABCDEF123456`
    }
};
```
### Storing config.js ConfigMap
```shell
kubectl create configmap nodejs-config --from-file=config.js -n production
```

### Storing config.js Secret
```shell
kubectl create secret generic nodejs-config --from-file=config.js -n production
```

### Mounting config.js file

```yaml
apiVersion: app/v1
kind: Deployment
metadata:
  name: nodejs-app
  namespace: production
spec:
  replicas: 8
  selector:
    matchLabels:
      app: nodejs-app
  template:
    metadata:
      labels:
        app: nodejs-app
    spec:
      containers:
      - name: nodejs-app
        image: nodejs-app:3.2.0
        ports:
          containerPort: 80
        volumeMounts:
        - name: nodejs-env-file
          mountPath: /app/config.js
          readOnly: true
    volumes:
    - name: nodejs-config-file
      secret:
        name: nodejs-config
```
