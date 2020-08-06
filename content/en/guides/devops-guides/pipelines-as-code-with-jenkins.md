---
title: "Pipelines as Code with Jenkins"
categories: devops
tags:
    - jenkins
date: "2020-07-01"
author: srainville
---

## Getting Started
Hello, world.

## Creating a Pipline Script
Place a file named `Jenkinsfile` inside the root of your project directory. 

```groovy
stage("test") {
    sh: "hello, world!"
}
```

