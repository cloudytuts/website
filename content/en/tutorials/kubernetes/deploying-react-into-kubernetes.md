---
layout: tutorial
title: Deploying React to Kubernetes
series: Up And Running on Kubernetes
categories: devops
weight: 60
tags:
  - nodejs
  - kubernetes
  - nginx
  - docker
date: "2020-06-30"
description: Learn how to containerize your React applications and deploy them into Kubernetes.
abstract: |
    Learn how to containerize your React applications and deploy them into Kubernetes.|
author: serainville
---

Containerization has seen exponential growth in popular since the advent of Docker. However, it wasn't long before people identified the challenges with orchestrating a containerized environment. While a number of solutions were developed, Kubernetes became king.

In this tutorial you will learn how to weld Kubernetes to run your React-based web applications.

Resources have been provided to aid you in following along with this tutorial. The resources as a repository in Github.

## Containerize your React App
Once transpiled, a React app will consist of at least two static files: an HTML file and a JavaScript file. Kubernetes is not capable of serving static files, as it is a container orchestrator. In order to server static files you must generate a container image capable of hosting your static files.

Nginx is very popular for serving static files. 

## Building Your Image
As with all NPM based projects, your project will need to be transpiled prior to be being deployed. When preparing your final container image us multistage Docker builds. 

The following Dockerfile example shows two stages. (1) the build stage transpiles (compiles) the source files and outputs your final static JavaScript file, (2) the final stage copies your newly generated static files to an Nginx-based image. 

```dockerfile
FROM node as build
COPY ./src .
RUN npm build

FROM nginx as final
COPY --from=build ./output/index.html /var/www/html
COPY --from=build ./output/myapp.js /var/www/html
```

{{< warning >}}
The example code snippet above does not define a `USER` action. By default a container will run with root privileges, therefore, it is an insecure image. [Setting a user]({{< relref path="tutorials/_index.md">}})
{{< /warning >}}

By using multistage builds we eliminate the need for placing our source files in the final image, which in most scenerios would be a security concern. Two stages are used in our build, and by doing so we separate our areas of concern. Our first stage compiles the project's static files, and the second stage generates the final image.

To build the project

```shell
  docker build -t my.docker.repo/myapp:v0.1.0 .
```

## Kubernetes Deployment Manifest
A Kubernetes deployment defines how an application will be deployed. Like all things Kubernetes, a manifest is typically a YAML file. 

```yaml
apiVersion: v1
kind: Deployment
metadata:
    name: react-app-deployment
    labels:
        kubernetes.io/name: reactapp-ui
        kubernetes.io/instance: reactapp-ui-staging
        kubernetes.io/component: UI
        kubernetes.io/verison: 1.0.1
spec:
  replicas: 3
  selector:
    matchLabels:
      app: react-app-deployment
  template:
    metadata:
      labels:
        app: react-app-deployment
    spec:
      containers:
        - name: react-app-ui
          image: my.docker.repo/reactapp-ui:v0.1.0
          ports:
            - containerPort: 80
```

### Metadata
Metadata is an important part of grouping common resources in Kubernetes. At minimum a deployment requires a `name` key, however, in larger, more complex Kubernetes environments you will need to expand your usage of metadata. 

{{< warning >}}
Never **deploy** a Docker container to production without adjusting the user.
{{< /warning >}}

Your success in implementing blue-green, canary, or other deployment strategies will depend on your ability separate and group common resources. 

{{< casestudy "Canary Deployments" >}}
With canary deployments its common for multiple versions of your applications to be deployed. Each version deploy could expose itself via the same service and, therefore, load balancer. By applying an appropriate metadata scheme it becomes easier to execution against a specific version of your application. 
{{< /casestudy >}}

The example deployment YAML file consists of four common labels.

| Metadata Label | Description |
| --- | --- |
| `kubernetes.io/name` | Useful for naming the component of your application. |
| `kubernetes.io/instance ` | is usefully for identifying the instance of your application, such as a specific environmental release. |
| `kubernetes.io/component` | explains what component your deployment belongs to (Database, cache, UI, frontend, backend, etc) |
| `kubernetes.io/version` | is self explanatory.  |


### Deployment Spec

The spec defines what container to deploy and how it will be deployed. It is here where you set the image to be deployed and whether there are replicas, for example. 



## Exposing React in Kubernetes

## ConfigMaps
Application configurations can differ from one environment to another. If all configurations were to be hard-coded into your application you would lose the ability of being portable; a single image of your application could not be deployed into any environment as environment specific images would be required. 

Kubernetes provides ConfigMaps, which is a key-value pair manifest for storing your applications configurations. 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: reactui-config
data:
    player_lives: 3
    ui_properties: "user-interface.properties"
    game.properties: |
        enemy.types=aliens,monsters
        player.maximum-lives=5
    user-interface.properties: |
        color.good=purple
        color.bad=yellow
        allow.textmode=true
```

Create a new file in the project directory named `.env`. The *.env* file, or environment file, allows you to set parameters specific for an environment. With Kubernetes, we can store the env file as a configMap. 

Create a new file named `.env` and add any parameters required for your environment.

```text
player.lives=5
```

The file can then be stored in Kubernetes as a configMap. 

```shell
kubectl create configmap
```

Mounting ConfigMaps into your Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: reactui-app
spec:
  containers:
  - name: ui
    image: cloudytuts.com/reactui-demo
    env:
      - name: PLAYER_INITIAL_LIVES
        valueFrom:
          configMapKeyRef:
            name: game-demo
            key: player_initial_lives
      - name: UI_PROPERTIES_FILE_NAME
        valueFrom:
          configMapKeyRef:
            name: game-demo
            key: ui_properties_file_name
```     
            
## Secrets
Protecting sensitive information is crucial to an application's security. Kubernetes provides Secrets as a means to store sensitive information in the cluster. The data is encrypted at rest as a base64 encoded string.

Kubernetes does not provide secrets lifecycle management, meaning out of the box you will be responsible for rotating secrets and ensuring pods have the updated information.

Aside from being encrypted, secrets are identical to ConfigMaps. They can be key-value pairs or files that can be read as environment variables or mounted as files.

### Creating Secrets Manifest
Create a new YAML file with the following structure. Your sensitive information will be stored under the `data` key, using a unique parameter name. 

{{ note }}
When using the `data` key, secrets must be ***base64*** encoded.
{{ /note }}

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: game-demo-secrets
data:
  db.user: ZGVtb2FwcF91c2Vy
  db.password: bXktc3VwZXItc2VjcmV0LXBhc3N3b3Jk
```

The values for `db.user` and `db.password` are base64 encoded. You can easily base64 encode a string on OSX and Linux using the base64 command.

```shell
echo -n 'super-secret-password' | base64
```

To apply the new secrets file, use the `kubectl apply` command.

```shell
kubectl apply -f demo-app-secrets.yaml
```

### Secrets as Environment Variables
With the secrets securely stored in Kubernetes you can now access them from a `pod` or `deployment`, for example. In our demonstration we will be accessing them for our React deployment.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: reactui-app
spec:
  containers:
  - name: ui
    image: cloudytuts.com/reactui-demo
    env:
      - name: PLAYER_INITIAL_LIVES
        valueFrom:
          configMapKeyRef:
            name: game-demo
            key: player_initial_lives
      - name: UI_PROPERTIES_FILE_NAME
        valueFrom:
          configMapKeyRef:
            name: game-demo
            key: ui_properties_file_name
      - name: DB_USER
        valueFrom:
          secretKeyRef:
            name: game-demo-secrets
            key: db.user
      - name: DB_PASSWORD
        valueFrom:
          secretKeyRef:
            name: game-demo-secrets
            key: db.password

```

We have combined both  `configMaps` and  `secrets` in our deployment. Our non-sensitive information is sourced from a configMap while the sensitive information from a secret.

## Accessing ConfigMap and Secrets
Both the ConfigMap data and the Secrets data are available as environment variables. Within your React application you will need to pull in the environment variables in order to use them. The variable names are the the `env` `name` values. For example, the database user enviroment is accessed using the `DB_USER` environment variable.

## Further Reading

The following key subject areas were included in this tutorial. 

* Namespaces
* Quotas
* Tags and Labels