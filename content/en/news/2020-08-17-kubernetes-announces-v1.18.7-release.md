---
title: "Kubernetes Announces v1.18.7 release"
date: 2020-08-17T13:26:48-04:00
draft: false
author: serainville
description: |
    Kubernetes has just released version v1.18.7 that moves the Kubernetes code base forward to Go 1.13.15 to address a number of minor issues.
tags:
    - kubernetes
---

Kubernetes has officially released a new minor version of Kubernetes, version 1.18.7. This new patch release doesn't add any new features or contain any bug fixes.

The patch release moves Kubernetes' code base to Go v1.13.15. That version of Go addresses a small set of issues relating to the `net/http` package and `cmd/go` package, as well as a few unrelated packages for Android and Apple. 

The `net/https` patch addresses a bug with `Server.ConnContext` accidentally modifying context for all connections. With Kubernetes handling a significant amount of `http` connections, this Go bug may have introduced problems in edge case scenarios with Kubernetes.

The `cmd/go` patch addresses a bug with a regression and a data race introduced in Go 1.13, which created a fatal error for concurrent map writes during go get.
