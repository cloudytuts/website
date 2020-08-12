---
title: "Pipelines as Code with Jenkins and Jenkinsfiles"
categories: devops
tags:
    - jenkins
date: "2020-07-01"
author: serainville
draft: false
description: |
    Learn how to use Jenkinsfiles in your project repositorys and implement pipelines as code with Jenkins
---

Introduce Jenkinsfiles to your project repositories to add continuous integration and continuous delivery pipelines in Jenkins.

* Create a new Jenkins Pipeline Project
* Configure the pipeline project to load a pipeline script from the cloned application project repository.

Over the years services such as CircleCI has grown popular with developers across the globe. The ease of including a build config file and controlling the pipelines empowered developers and promited Agile workflows. 

While SasS solutions provided by CircleCI, Bitbucket, and Github, for example, all allow developers to easily define pipelines, the same can be easily achieved with Jenkins. 

Jenkins pipeline scripts are handy addition to Jenkins that allow us to write what has been coined as pipelines-as-code. By defining your pipeline jobs as groovy scripts they can be version controlled.

By combining a Jenkins pipeline scripts, Jenkinsfile, with a project repository the pipeline can evolve with the project itself. 

## Create a Jenkins Project
In order to run pipelines as code a Jenkins project must be created first.
1. Create a new Pipeline Project.
1. Set the Git repository for your application project.
1. Set the location of your Jenkinsfile in the application repository.

## Create your first pipeline
Place a file named `Jenkinsfile` inside the root of your project directory. 

Jenkinsfile pipeline scripts are written in the **Groovy** language. The following is an example of a very basic pipeline that tests and builds a NodeJS project.

```groovy
node {
    // Run unit tests to validate feature functionality
    stage("test") {
        sh: "npm lint"
        sh: "npm test"
    }

    // Compile or build the application for release
    stage("build") {
        sh: "npm run-script build-production"
    }
}
```
### Node
Node is used to match a pipeline against Jenkins nodes. For instance, a group of nodes can be labelled with "docker" to identify them as having all the necessary packages to run and build Docker images. By setting the node as `node "docker" {` we can instruct Jenkins to only execute our pipeline on hosts that are running Docker.

### Stage
Pipeline stages are defined to separate different stages of a pipeline. The most typical stages are "test" and "build", however, these can be given any arbitrary name. The primary puprose of stages is to organized steps into groups, which is very handy when following a job or troubleshooting an error.

### Steps
Steps are defined as the actions that are performed in a pipeline. In the example above, we have a number of `sh` actions, which are shell executions. Jenkins has an extensive library of built-in actions.

## Pipelines as Code
The largest benefit of a groovy-based pipeline script is that it empowers you to add custom logic. While Jenkins provides a number of built-in functions, you're likely to run into scenarios where custom logic is required.

For example, you may be required to create releases of React applications. A typical use case is to archive the compiled output of a React or any JavaScript based application into a compressed tar ball. 

```groovy
function createReleaseFile(String source, String target) {
    sh: 'tar -zxf ${target}.tar.gz ${source}'
}
```

Once defined in our pipeline we can execute the function from our pipeline stages.

```groovy
stage("release") {
    createReleaseFile("output/", "release")
}
```

## Vars


