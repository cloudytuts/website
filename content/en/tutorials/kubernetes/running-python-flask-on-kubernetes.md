---
kind: tutorials
layout: tutorial
title: Running Python Flask on Kubernetes
series: Kubernetes in Action
date: "2020-07-06"
tags: 
    - python 
    - flask 
    - kubernetes
    - docker
git_repo: "https://github.com/cloudytuts/kubernetes-in-action.git"
description: Learn how to containerize and deploy your Python Flask application in a production Kubernetes cluster, as well as how to package your application in Helm.
author: serainville
---

Containerizing Python Flask applications and running them in production on Kubernetes is not a trivial task. While the steps of building a Docker image and running the image in Kubernetes may seem simply, your approach must be well thought out to ensure reliability and security.

In this tutorial you will learn how to containerize your Flask applications. You will also learn how to apply configurations separately from your Docker image, and how to handle secrets to avoid storing sensitive information in your application's project repository.

## Topics Covered
* Containerize your Python Flask Application
* Deploy Python Flask in Kubernetes
* Config management for Python Flask in Kubernetes
* Secrets management for Python Flask in Kubernetes
* Packaging Flask Applications with Helm

## Getting Started
### Prerequisites
* Docker [installed](https://docs.docker.com/engine/install/) on your development machine.
* A working Kubernetes cluster.

### Resources
* Kubernetes resource files used in this tutorial are available in the [CloudyTuts Github repository](http://github.com/cloudytuts/kubernetes-in-action)
* A Python Flask application. A [demo app]({{<param git_repo>}}) is available for those who do not have one and want to follow along.

## Preparing Flask for Dockerization
### Application Server
There is a misconception that Flask can be run directly via it's development server once your application is containerized. I strongly discourage this idea, as the development server is not purpose built for production traffic. 

If you have not added an application server to your project, we strongly advised you do that prior to building a production facing image of your application. In our demonstration app we have added `gunicorn`.


```text {hl_lines=[7]}
click==7.1.2
Flask==1.1.2
itsdangerous==1.1.0
Jinja2==2.11.2
MarkupSafe==1.1.1
Werkzeug==1.0.1
Gunicorn==20.0.4
```

### Flask Configs for Containers
A configuration file for Python Flask defines important values for your application depending on the environment it is deployed to. An example `config.py` file is provided below. Notice how values typically associated with the different environments has been moved to the parent *Config* class.

```python {linenos=table, hl_lines=["6-9"]}
import os

class Config(object):
    DEBUG = False
    TESTING = False
    DB_NAME = os.environ.get("DB_NAME")
    DB_USERNAME = os.environ.get("DB_USERNAME")
    DB_PASSWORD = os.environ.get("DB_PASSWORD")
    SECRET_KEY = os.environ.get("SECRET_KEY")
    SESSION_COOKIE_SECURE = True

    if not DB_NAME or not DB_USERNAME or not DB_PASSWORD:
        raise ValueError("No database config set for Flask application")
    if not SECRET_KEY:
        raise ValueError("No SECRET_KEY set for Flask application")

class ProductionConfig(Config):
    pass

class DevelopmentConfig(Config):
    DEBUG = True
    SESSION_COOKIE_SECURE = False

class TestingConfig(Config):
    TESTING = True
    SESSION_COOKIE_SECURE = False
```

Highlighted are values used to set parameters for the database. These values are typically defined per environment rather than in the inherited parent class *Config*. With Docker and Kubernetes these values will be defined when the application is deployed.

Since the application shouldn't run without database information or the `SECRET_KEY` defined, we've added extra logic to force the application to fault if key variables are not set. For `SECRET_KEY` this ensure session security no matter the environment our application is deployed into.

## Dockerizing your Flask Apps
### Building Docker Images
#### Selecting a Base Image
When building your own image you will need to base it off a base image. There are a number of options available, from native Ubuntu or Alpine Linux to fully prepared Flask. Our recommendation is to keep things as simple and lightweight as possible, and to find use official images where possible.

For Python Flask it is recommended to use `python` as your base image. It is also recommended to be explicit with which release of the image by specifying a tag. In our example Dockerfile, we've chosen `python:3.8.3-alpine` as our base image.

{{< note >}}
Always be as explicit as possible when setting your base image. Never use the *latest* tag and instead choose a specific version. Latest truly means latest, which means every build could likely use a different base image, which may be problematic when your development, testing, and production environments end up with a wild variety of different Python versions.
{{< /note >}}

For our example we've chosen to run Python 3.8.3 and to use the smallest image available. Our base image will be `python:3.8.3-alpine`.

#### Dockerfile
A *Dockerfile* is a set of instructions to build a Docker image using well defined actions, such as `FROM`, `COPY`, `RUN`, `ENTRYPOINT`, `CMD`, `ENV`. Creating a Dockerfile is the first step towards dockerizing your application.

The following is an example dockerfile used to generate a Docker image for a Python Flask image. The same file is used in the Git repository containing our example demoapp. 

```dockerfile
FROM python:3.8.3-alpine

COPY demoapp .

RUN pip install -r requirements.txt

ENTRYPOINT ["flask", "run --host=0.0.0.0"]
```

To understand what is happening in the Dockerfile above, the following is a breakdown of the actions used.

| <br> | <br> |
|:-------|:----------- |
| **FROM**   | The base image used to build our application's image |
| **COPY**   | Copies the contents of our application directory on our local machine into the Docker image. |
| **RUN**    | Executes `pip install -r requirements.txt` during Docker build  to ensure dependencies are installed. |
| **CMD**    | Instructs Docker to execute `gunicorn --bind=0.0.0.0:5000 demoapp:app` when a container is started based on the image being built. |

{{< note >}}
Every Dockerfile action creates a new container layer. Each layer is cached so that subsequent builds can be sped up. Always order your actions such that more frequently modified actions are placed near the end, while actions with fewer changes are at the top. 
Layers below one that changes will have their caches invalidated, forcing their actions to be executed. This usually causes unnecessary build times.
{{< /note >}}


For reference, the demoapp consists of the following project directory structure.
```
.
├── demoapp
│   ├── app
│   │   ├── __init__.py
│   │   └── config.py
│   ├── run.py
│   └── requirements.tzt
└── Dockerfile
```


{{< warning >}}
Every Dockerfile action creates a new container layer. Each layer is cached so that subsequent builds can be sped up. Always order your actions such that more frequently modified actions are placed near the end, while actions with fewer changes are at the top. 
Layers below one that changes will have their caches invalidated, forcing their actions to be executed. This usually causes unnecessary build times.
{{< /warning >}}



#### Building an Image
To build Docker images from a Dockerfile, the `docker build` command is executed. In the example below we've added `-t myproject:v1.0.0` to tag the image, giving it a name that will be referenced when running a container based on it. The '`.`' at the end instructed docker build to load a Dockerfile from the current directory.

```shell
docker build -t myproject:v1.0.0 .
```

As the image is built information about each step or layer will be outputted to your screen. 


#### Environment Variables
The `config.py` file used to configure the Flask application sets values using environment variables. These values can be set in a Kubernetes Deployment manifest, which will be covered later.

#### Testing Your Docker Image
To run your newly generated Docker image use the `docker run` command. As mentioned above, the Python Flask application expects certain environment variables to be set. It also exposes itself on port 5000.

```shell
docker run -d -p 5000:5000 \
-e FLASK_ENV=development \
-e DB_NAME=demoapp_dev \
-e DB_HOST=mysql01.local \
-e DB_USERNAME=demoapp_dev \
-e DB_PASSWORD=P@ssw0rd1 \
demoapp:1.0.0
```

The `docker run` command above uses the `-e` flag to set environment variables. For non-sensitive information this is acceptable, but notice that we are also entering our database password in clear text. Unfortunately, this is a limitation of vanilla Docker. Docker Compose and Kubernetes both have a way of better handling sensitive information, which will be discussed later. 

To verify the container launched successfully and your Flask application is accepting connections, use the `docker ps` command. The command will output a list of all running containers.

```shell
docker ps
```
    | OUTPUT
```shell
REPOSITORY                                                       TAG                                              IMAGE ID            CREATED             SIZE
serainville/flask-demo                                           1.0.0                                            7cc67ff28d70        19 hours ago        959MB
serainville/flaskr                                               1.0.0                                            2616ac211915        23 hours ago        944
```

When your new container is started its container ID will be generated and printed to your console. Take note of this ID, as you will need it for managing your container and troubleshooting.

### Troubleshooting failed containers
If you do not see your container running it likely failed and exited unexpectedly. To see a list of all containers use the `docker ps -a` command. Once you have found the container view its logs using the `docker logs <container-id>` command.

```
docker logs aef01
```

An example of logs generated by a failed Python Flask container may look something like the following. Notice in this example the Flask container failed to launch because an environment variable wasn't set for SECRET_KEY.

```shell
failed example
```


## Deploying to Kubernetes
A Pod is the smallest construct of a Kubernetes resource. A pod can consist of one or more containers that work together to provide a single service.

To deploy an application into Kubernetes you will need to define either a deployment or a pod manifest. Deployment manifests are strongly recommended, as they can control replica scaling as well as scheduling failed pods.

We will create the following manifests to deploy our Python Flask application:
* Deployment Manifest
* Service Manifest

In addition, we will need to create the following support manifests to hold our configuration data for each environment, as well as secrets for sensitive information.
* ConfigMap manifest, one per environment (development, testing, production)
* Secrets manifest, one per environment (development, testing, production)





## Helm Charts (Optional)
So far we have had to manage each Kubernetes resource used by our Flask application manually. As the number of release environments grow, so do the number of resources needing to be maintained. Helm answers that problem by providing a means to template everything.



### Installing Helm
On OSX helm can be installed via brew.
```shell
brew install helm
```

### Generating a Helm Chart
To generate the scaffolding of your Helm package, use the `helm create <chart-name>` command. For example, we'll generate a chart for demoapp.

```shell
helm create demoapp
```

### Installing Your Chart
To deploy your application in Kubernetes will perform a `helm install`. The command accepts two arguments, the release name and the Helm chart to use. 

```
helm install <release-name> <chart>
```

For example, to create a release named *demoapp-dev* using our *demoapp* chart run the following command.

```shell
helm install demoapp-dev demoapp
```




    | OUTPUT
```shell
NAME: demoapp-dev
LAST DEPLOYED: Fri Jan 31 18:08:35 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: none
```


