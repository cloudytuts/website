---
title: "How to Run AngularJS on Kubernetes"
date: 2020-08-21T15:58:25-04:00
draft: true
author: serainville
tags:
    - kubernetes
    - angularjs
    - docker
description: |
    Learn how to containerize your AngularJS applications and run them on Kubernetes in a production environment.
---


## What's Covered
* Learn how to use multistage Docker builds
* Learn how to containerize AngularJS applications
* Learn how to run AngularJS apps in Kubernetes
* Learn how to use secrets guard sensitive information
* Learn how to use configMaps to store application configs

## Containerize AngularJS
AngularJS applications are transpiled into static files. As such we need a web server to serve our Angular applications, such as NGINX or Apache Web Server. In this guide we will use NGINX as it has benefits over Apache for static content.

To simply our build and release process we will construct a multistage Dockerfile. This strategy allows us to build our Angular application, with all required build and test tools, and generate a final, light-weight Docker image. 

### AngularJS Dockerfile
As with all node-based applications, suchas Angular and React, we must build a final artifact using Node tooling. This makes our decision for build stage image easy. Rather than building a container with Node installed, we can pull down the official Node image.

{{< note >}}
Always use a specific node version when developing and building. Even bug fixes can introduce build differences that may affect your final artifacts.
{{< /note >}}

#### Build Stage
Create a new file named `Dockerfile` and add the following contents. The Dockerfile in this example uses Node version 14.8.0. It copies the local `src` directory into a directory in the build container with the same name, and then runs a standard `npm build` command.

```dockerfile {hl_lines=["1-7"]}
FROM node:14.8.0 AS build

COPY ./src ./src

RUN npm install \
    & npm run-script test
    & npm run-script build

FROM nginx AS final
COPY --from=build build/* /usr/local/nginx/html
```

#### Final Stage
The final stage is what produces our final Docker image. Since we've decided osing NGINX in this tutorial, we will use an `nginx` image as our base.

```dockerfile {hl_lines=["9-10"]}
FROM node:14.8.0 AS build

COPY ./src ./src

RUN npm install \
    & npm run-script test
    & npm run-script build

FROM nginx AS final
COPY --from=build build/* /usr/local/nginx/html
```






