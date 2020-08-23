---
title: "How to containerize Angular, React, and Vuejs Apps with Docker"
date: 2020-08-22T15:18:48-04:00
draft: false
author: serainville
tags:
    - angular
    - react
    - vue
    - docker
description: |
    Learn how to build JavaScript based frameworks Angular, React, and VueJS in Docker for running in a highly containerized environment, such DockerSwarm and Kubernetes.
---


## What's Covered
* Containerizing Angular, React, and VueJS.
* Mutlistage Docker builds.
* Use NGINX to serve Angular, React and VueJS applications.
* Using environment variables to configure applications
* Using .env files for configuring applications.


## Environment Agnostic Configurations
When designing the container image for your JavaScript application you should keep it as simple as possible. The image should not be specific to any environment, rather it should be able to operate in any environment. 

Building environment specific images as a workflow dependancy where changes to one environment must be promoted to other environments. As this is usually never a simple operation, since environments rarely mirror each other, you will begin to experience config drift.

Config drifts bring risk into your development workflows. If there is too much drift an application's behavoir may change between environments, which will invaldidate your test results. 

### Environment Variables
Environment configuration should be injected using environment variables where possible. When environment variables are not appropriate, a deployment pipeline should then be used to inject configuration files.

### .ENV Files
An  alternative to environment variables for node-based projects is `.env` files. Using files for configuring your applications is not as flexible as an environment variable. Your application requires access to the file during run-time, but the file should never be part of the build image for security reasons.

An `.env` file must be injected into the running container image and the most common way of accomplishing this is through volumes.

## Docker Build
### Multistages
When building a Docker image for any node-base JavaScript project you will have to decide where to compile your application. The two options are build the application before your Docker build, or use multistage Docker builds that build your application.

The problem with node-based JavaScript applications is they require two things that do not belong in production. Node projects require build tools as well as a mammoth node_modules dependancy directory. Neither of which should exist in a production image.

### Two stage build
A two stage build is used strictly to compile, or transpile in the case of JavaScript, your application. We install all of your dependant tools and project dependancies, and build the application. 

```dockerfile
FROM node:latest AS build
COPY ./src .
RUN npm install \
    & npm run-script build-production
```

The second stage creates the final Docker image. This image will only contain our application's build artifacts and services required to run the application.

```dockerfile
FROM nginx:latest AS final
COPY --from=build build/* /usr/local/nginx/html
```

### Three Stage Build
An example of a three stage build would test, compile, and build a final application container. We let our Docker container handle most of the continuous deployment pipeline. 
* We ensure our unit tests pass
* We build our application artifacts
* We output a final image for an environment agnostic image







