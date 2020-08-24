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
Once transpiled, a React app will consist of at least two static files: an HTML file and a JavaScript file. Kubernetes is not capable of serving static files, as it is a container orchestrator. In order to server static files you must generate a container image capable of hosting your static files.

Nginx is very popular for serving static files. 

## Building Your Image
As with all NPM based projects, your project will need to be transpiled prior to be being deployed. When preparing your final container image us multistage Docker builds. 

The following Dockerfile example shows two stages. (1) the build stage transpiles (compiles) the source files and outputs your final static JavaScript file, (2) the final stage copies your newly generated static files to an Nginx-based image. 

```dockerfile
# Build stage with all source files, dependancies, and necessary build tools
FROM node as build
COPY ./src .
RUN npm run-script build

# Clean, light-weight final image with only build artifacts
FROM nginx:1.17 as final
COPY --from=build build /usr/share/nginx/html
```

{{< warning >}}
The example code snippet above does not define a `USER` action. By default a container will run with root privileges, therefore, it is an insecure image. [Setting a user]({{< relref path="tutorials/_index.md">}})
{{< /warning >}}

By using multistage builds we eliminate the need for placing our source files in the final image, which in most scenerios would be a security concern. Two stages are used in our build, and by doing so we separate our areas of concern. Our first stage compiles the project's static files, and the second stage generates the final image.

The command below builds the Docker image based on our `Dockerfile`. The `-t` flag tags the image with the Docker repository name and our app's name. It also includes a version to easily identify the build.

```shell
  docker build -t my.docker.repo/myapp:v0.1.0 .
```

If you are using a remote Docker repistory, you will need to push your newly created image to it.

```shell
docker push my.docker.repo/myapp:v0.1.0
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

## ConfigMaps
Application configurations can differ from one environment to another. If all configurations were to be hard-coded into your application you would lose the ability of being portable; a single image of your application could not be deployed into any environment as environment specific images would be required. 

Kubernetes provides ConfigMaps, which is a key-value pair manifest for storing your applications configurations. 

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
    name: reactui-app
data:
    api.server: https://api.server:3000
    cards.screen: 10
```

Create a new file in the project directory named `.env`. The *.env* file, or environment file, allows you to set parameters specific for an environment. With Kubernetes, we can store the env file as a configMap. 

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

Using the `kubectl get secrets` command we can see the state of our newly created configMap resource in Kubernetes. The output has three columns: `NAME`, `DATA`, and `AGE`. 

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
            
## Secrets
Protecting sensitive information is crucial to an application's security. Kubernetes provides Secrets as a means to store sensitive information in the cluster. The data is encrypted at rest as a base64 encoded string.

React applications typically use api keys to interact with backend services and third-party services. API keys are a good example of a secret, as it is usually not desirable for it to be exposed to the public. Rather than commiting your API key in your source code, creating a potential security risk, use a Kubernetes secret and pull it into your React application as an environment variable.

Kubernetes does not provide secrets lifecycle management, meaning out of the box you will be responsible for rotating secrets and ensuring pods have the updated information.

Aside from being encrypted, secrets are identical to ConfigMaps. They can be key-value pairs or files that can be read as environment variables or mounted as files.

### Creating Secrets Manifest
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