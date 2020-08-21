---
title: "How to Dockerize React Apps With Multistage Builds"
date: 2020-08-20T21:33:32-04:00
draft: false
author: serainville
tags:
    - docker
    - react
description: |
    Learn how to use Docker multistage builds to build images for React applications.
---

In this tutorial, you will learn how to use Docker's multistage builds in order to create a small, lightweight image for your React applications. 

React applications and the node module dependancies must be transpiled 

## Multistage Docker Builds
Previously, a Docker build consisted only of a single stage. All artifacts from the command execute to build to image remained in the final image, which is undersiable for any project that must compiled or transpiled. To workaround this limitation most applications were compiled into artifacts, which were then copied to the Docker image during a new image build.

With multistage builds we can now create multiple stages to perform tasks.

Stages are created everytime there is a `FROM` command. An optional name can be given to a stage using `AS <stage-name>`. An example of a stage in a Dockerfile looks like the following example

```dockerfile
FROM <image>:<tag> AS <stage-name>
```

Each stage creates an independant container layer. Files can be copied from earlier stages into later stages, and the final stage is used to build the final image.

```dockerfile
FROM <image>:tag> AS build
RUN some-build-command

FROM <image>:<tag> AS tests
RUN test-command-1
RUN test-command-2

FROM <image>:<tag> AS final
COPY --from=build <source-artifact> <target-artifact>
```


### Node Modules
Node-based projects usually rely on a number of dependancies. Most dependancies are sourced from the NPM public package repository, which are stored in the `node_modules` directory. The files in this directory are required to build the dependancies into your apps final artifact, but it has no place on the production application server.

Multistage Docker builds allow us to build our React applications as part of our Docker iamge build, without keeping large, unnecessary directories like `node_modules`.

## Containering React Application

```dockerfile
FROM node:latest AS test
COPY ./src ./src
RUN npm install \ 
    & npm run-script test

FROM node:latest as build
COPY --from=build ./src ./src
RUN npm run-script build

FROM nginx:latest AS final
COPY --from=build ./build/* /usr/local/nginx/html
```






