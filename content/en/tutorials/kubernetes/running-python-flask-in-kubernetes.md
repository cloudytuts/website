---
kind: tutorials
layout: tutorial
title: Running Python Flask in Kubernetes
series: Kubernetes in Action
date: "2020-07-06"
tags: python flask kubernetes docker
git_repo: "https://github.com/cloudytuts/kubernetes-in-action.git"
abstract: Learn how to containerize and deploy your Python Flask application in a production Kubernetes cluster, as well as how to package your application in Helm.
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

* A Python Flask application. A [demo app]({{<param git_repo>}}) is available for those who do not have one and want to follow along.

## Preparing Flask for Dockerization
### Applicaiton Server
There is a misconception that Flask can be run directly via it's development server once your application is containerized. I strongly discourage this idea, as the develpoment server is not purpose built for production traffic. 

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

### Flask Config
A configuration file for Python Flask typically has configuration values for each environment the application will run in. 

```python
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


## Dockerizing your Flask Apps



### Building Docker Images
When deciding on building a Docker image for your Flask application, choose your base package wisely. In this tutorial the base will be an official Python image (`python:3.8.3-alpine`) as a secure starting point. 

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

The project's `Dockerfile` has the following contents. Following today's best practices we've chosen an official Python image based on Alpine linux. Our image should have a minimal footprint with exactly one specific role: running our python-based application. 

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
| **COPY**   | Copies the contents of our application directory into the Docker image. |
| **RUN**    | Executes `pip install -r requirements.txt` during Docker build  to ensure dependencies are installed. |
| **CMD**    | Instructs Docker to execute `gunicorn --bind=0.0.0.0:5000 demoapp:app` when a container is started based on the image being built. |


#### Building an Image
To build a Docker images from a Dockerfile, the `docker build` command is executed. In the example below we've added `-t myproject:v1.0.0` to tag the image, giving it a name that will be referenced when running a container based on it. The '`.`' at the end instructed docker build to load a Dockerfile from the current directory.

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
-e DB_NAME=demoapp_dev \
-e DB_HOST=mysql01.local \
-e DB_USERNAME=demoapp_dev \
-e DB_PASSWORD=P@ssw0rd1 \
demoapp:1.0.0
```

The `docker run` command above uses the `-e` flag to set environment variables. For non-sensitive information this is acceptable, but notice that we are also enteirng our database password in clear text. Unfortunately, this is a limitation of vanilla Docker. Docker Compose and Kubernetes both have a way of better handling sensitive information, which will be discussed later. 

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

## Helm Charts (Optional)
So far in the tutorial we've deployed all of the resources required for our application by hand. Helm is used to generate packages of our application's for Kubernetes and manage releases. 

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
To deploy your application in Kubernetes will perform a `helm install`.

```shell
helm install demo-guestbook guestbook
```
    | OUTPUT
```shell
NAME: demo-guestbook
LAST DEPLOYED: Fri Jan 31 18:08:35 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: none
```


