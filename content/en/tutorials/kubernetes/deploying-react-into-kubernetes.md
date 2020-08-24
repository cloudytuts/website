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
description: |
    Learn how to containerize your React applications and deploy them into Kubernetes, and pull configuration data from configMaps and secrets.
author: serainville
repo: https://github.com/cloudytuts/kubernetes-in-action
---

Containerization has seen exponential growth in popular since the advent of Docker. However, it wasn't long before people identified the challenges with orchestrating a containerized environment. While a number of solutions were developed, Kubernetes became king.

In this tutorial you will learn how to weld Kubernetes to run your React-based web applications.

Resources have been provided to aid you in following along with this tutorial. The resources as a repository in Github.

## Containerize your React App
Once transpiled, a React app consists of one or more static files. Kubernetes is not capable of serving static files alone, as it is a container orchestration platform. In order to server static files you must generate a container image capable of hosting your static files.

A very popular web server for serving static files is NGINX. In this tutorial we will adopt it by building our image using the official NGINX image.

### Building Your Image
To build a Docker image you must create a `Dockerfile`, which defines actions to create a container image. To streamline our processes we will create a Dockerfile that combines our application build with application deployment. 

Our `build` stage will contain all of the tooling, node modules, and source files necessary to build our React app. There's no need to optimize this stage as its existence is only temporary.
  * Use the official Node docker image as a base
  * Use a specific verison of node that matches our code base (14.8.0)
  * Use a Debian based image to allow easier customization

Our `final` stage will build our final container image. As this is a React app we need a static file web server to host our application.
  * Build image based on official NGINX Docker image
  * Copy build artifacts from build stage to NGINX root document directory

Based on our requirements above the final Dockerfile would resemble the following.

```dockerfile
# Build stage for compiling the React app
FROM node:14.8.0-stretch as build
RUN mkdir /app
COPY . /app
WORKDIR /app
RUN npm install \
    && npm run-script build \
    && ls -l

# Final stage for creating the final Docker image
FROM nginx:1.19-alpine as final
COPY --from=build /app/build/ /var/www/html
```

{{< note >}}
The Dockerfile above pulls a very specific version of Node. It is strongly recommended to match your Node Docker version with the one used in development. Even a bug release may introduce variances that affect how your application operates.
{{< /note >}}

To build our Docker image you use the `docker build` command.  The `-t` flag tags the image to more easily identify the image, and we will use it to reference our apps name and its release version.

```shell
  docker build -t myapp:v0.1.0 .
```

If your Kubernetes cluster is hosted and not running locally, you will need to publish your image to a Docker repository accessible to your cluster. For example, Docker Hub is publicly available and any cluster is able to access it. If you are running in Google Cloud, you could push your image to Google Container Repository (GCR).

```shell
docker push my.docker.repo/myapp:v0.1.0
```

## Deployment
A Kubernetes deployment defines how an application will be deployed. Like all things Kubernetes, a manifest is typically a YAML file. 

Create a new file named `deployments.yaml` and add the following contents.

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

To create the deployment resource in your Kubernetes cluster use the `kubectl apply` command.

```shell
kubectl apply -f deployment.yaml
```

### Metadata
Metadata is an important part of grouping common resources in Kubernetes. Every resource in a Kubernetes cluster will have a name key. At minimum a deployment requires a `name` key, however, in larger, more complex Kubernetes environments you will need to expand your usage of metadata. 

Your success in implementing deployment strategies such as blue-green, and canary will depend on your ability to separate and group resources. Kubernetes provides a best practice for labelling applications. We've included the most common labels that most applications should use.

| Metadata Label | Description |
| --- | --- |
| `kubernetes.io/name` | Useful for naming the component of your application. |
| `kubernetes.io/instance ` | is usefully for identifying the instance of your application, such as a specific environmental release. |
| `kubernetes.io/component` | explains what component your deployment belongs to (Database, cache, UI, frontend, backend, etc) |
| `kubernetes.io/version` | is self explanatory.  |


#### Spec

The spec defines what container to deploy and how it will be deployed. It is here where you set the image to be deployed and whether there are replicas, for example. 

The `replicas` key-value sets the number of Pods the deployment will create for your application.
The `template` key is used to define the containers that will run in your pods.

## Storing Configurations
Most applications require environment specific configurations. These configurations can range from the number of cards to display on screen, to connections for a backend system.

There are two types of resources in Kuberentes for storing configuration data. The first is a `ConfigMap` and the second is a `Secret`. 

**ConfigMaps** are for general configurations that are not sensitive. A typically use case is backend hostname of your database server, or how many cards to display on your screen.

**Secrets** are designed to store sensitive information. Private keys, API keys, and user credentials are typically stored in these. Kuberentes protects secrets by encrypting them on disk. As another level of protection, values are base64 encoded to protect from people looking over your shoulder.

### ConfigMaps
Application configurations can differ from one environment to another. If all configurations were to be hard-coded into your application you would lose the ability of being portable; a single image of your application could not be deployed into any environment as environment specific images would be required. 

Kubernetes provides ConfigMaps, which is a key-value pair manifest for storing your application's  non-sensitive configuration parameters. 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: reactui-app
data:
    api.server: https://api.server:3000
    cards.screen: 10
```
#### Files
ConfigMaps can also be used to store files. These files can then be mounted in a Pod for an application to read it. For example, Node projects can use an `.env` to store environment configs. The `.env` could be stored in a ConfigMap, mounted to a running Pod, and read by your application at startup.

Create a new file named `.env` and add any parameters required for your environment.

```text
api.server=https://api.server:3000
```

The file can then be stored in Kubernetes as a configMap. 

```shell
kubectl create configmap reactui --from-file=.env
```
  | Output
```shell
configmap/reactui created
```

Using the `kubectl get configmap` command we can see the state of our newly created configMap resource in Kubernetes. The output has three columns: `NAME`, `DATA`, and `AGE`. 

```shell
kubectl get configmap
```
  | Output
```shell
NAME               DATA   AGE
reactui            1      99s
```

A brief description of each column:
* The `NAME` column is the name we've given our configmap.
* The `DATA` column is the number of parameters, or keys in the configmap. We've only created a single parameter: `api.server`.
* The `AGE` column shows how long the resource has existed.

We can view much more inforamtion about our configMap using the `kubectl describe configmap` command. 

```shell
kubectl describe configmap reactui
```
  | Output
```shell
Name:         reactui
Namespace:    default
Labels:       <none>
Annotations:  <none>

Data
====
.env:
----
api.server=https://api.server:3000
```

### Environment Variables from ConfigMaps
The most basic way to use configMaps is to create environment variables from the keys. To create environment variables in your pods based on values in a ConfigMap, you will define the environment variable name and which key value to use from a ConfigMap.

```yaml {hl_lines=["9-19"]}
apiVersion: v1
kind: Pod
metadata:
  name: reactui-app
spec:
  containers:
  - name: ui
    image: cloudytuts.com/reactui-demo
    env:
      - name: API_SERVER
        valueFrom:
          configMapKeyRef:
            name: reactui-app
            key: api.server
      - name: CARDS_PER_SCREEN
        valueFrom:
          configMapKeyRef:
            name: reactui-app
            key: cards.screen
```     
            
The example above highlights the changes necessary to create environment variables in your deployment. We've defined two environment variables: `API_SERVER` and `CARDS_PER_SCREEN`.

Both environment variables use a configMap named `reactui-app`. The `API_SERVER` gets its value from the `api.server` configmap key, and the `CARDS_PER_SCREEN` gets its value from the `cards.screen` configmap key.

Alternatively, the entire configMap can be used to create environment variables, rather than specifying individual keys.

```yaml
envFrom:
  configMapRef:
    name: reactui-app
```

With this method the environment variables will be named after the configmap key.


### Secrets
Protecting sensitive information is crucial to an application's security. Kubernetes provides Secrets as a means to store sensitive information in the cluster. The data is encrypted at rest as a base64 encoded string.

React applications typically use api keys to interact with backend services and third-party services. API keys are a good example of a secret, as it is usually not desirable for it to be exposed to the public. Rather than commiting your API key in your source code, creating a potential security risk, use a Kubernetes secret and pull it into your React application as an environment variable.

Kubernetes does not provide secrets lifecycle management, meaning out of the box you will be responsible for rotating secrets and ensuring pods have the updated information.

Aside from being encrypted, secrets are identical to ConfigMaps. They can be key-value pairs or files that can be read as environment variables or mounted as files.

#### Creating Secrets Manifest
Create a new YAML file with the following structure. Your sensitive information will be stored under the `data` key, using a unique parameter name. 

{{< note >}}
When using the `data` key, secrets must be ***base64*** encoded.
{{< /note >}}

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: game-demo-secrets
data:
  api.key: bXktc3VwZXItc2VjcmV0LWFwaS1rZXk=
```

The value for `api.key` id base64 encoded. You can easily base64 encode a string on OSX and Linux using the base64 command.

```shell
echo -n 'my-super-secret-api-key' | base64
```

To apply the new secrets file, use the `kubectl apply` command.

```shell
kubectl apply -f demo-app-secrets.yaml
```

#### Secrets as Environment Variables
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
      - name: API_SERVER
        valueFrom:
          configMapKeyRef:
            name: reactui-app
            key: api.server
      - name: CARDS_PER_SCREEN
        valueFrom:
          configMapKeyRef:
            name: reactui-app
            key: cards.screen
      - name: API_KEY
        valueFrom:
          secretKeyRef:
            name: reactui-secrets
            key: api.key          

```

We have combined both  `configMaps` and  `secrets` in our deployment. Our non-sensitive information is sourced from a configMap while the sensitive information from a secret.

## Accessing ConfigMap and Secrets in React
Both the ConfigMap data and the Secrets data are available as environment variables. Within your React application you will need to pull in the environment variables in order to use them. 

Pulling environment variables into your React application is done using `process.env.VARIABLE_NAME`. In the example code snippet below, we are limiting the number of cards display on a screen using the `CARDS_PER_SCREEN` environment variable, set in our configMap as `cards.screen`

```javascript
render() {
  return (
    <div>
      <cards limit={process.env.CARDS_PER_SCREEN} />
    </div>
  );
}
```

## Exposing React in Kubernetes
The final step for your React app is to expose through a Kubernetes service. While you could expose a pod directly to traffic, the lifecycle of pods is typically very short. A service, on the hand, has a much longer, almost permanent lifecycle. 

Services also allow load balancing traffic between a cluster of pods.

Services are created using a service manifest. the following example serves your React app through a load balancer.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: game-demo
spec:
  type: loadBalancer
  selector:
    app: game-demo
  ports:
    - protocol: TCP
      port: 80
```



## Further Reading

The following key subject areas were included in this tutorial. 

* Namespaces
* Quotas
* Tags and Labels