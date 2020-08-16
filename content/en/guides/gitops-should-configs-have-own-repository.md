---
title: "GitOps: Should Configs be Stored Separately from Application Code"
date: 2020-08-15T21:24:51-04:00
draft: false
author: serainville
tags:
    - gitops
description: |
    Should configs be stored in the same repository as code or should they be separated into different repositories.
---

In the era of git an age a common question developers and SREs ask themselves is "where should I store my configs?" Should configurations be stored in an application's repository or should it be stored separately? 

With the introduction of **GitOps** this question will be asked more frequently. GitOps desires to achieve a goal of having all operations done through version control systems. GitOps implementations can be more difficult in some environment depending on the config strategy employed. 

## Understanding Lifecycles
The reality is configurations and applicaiton source code has very different lifecycles. An application typically has frequent code commits to add features, fix bugs, and address technical debt. Configurations, on the other hand are rarely updated.

Configurations tend to be static in nature. Most features won't require a configuration update, and

## Environment Branches
One strategy some teams use is to carve separate environment branches. While this may seems easy to maintain at first, configuration variance starts to creep into the repository. The reason for this is simple: application code changes will be merged into the environment branches as a means to promote it up the ladder, however, an environment's configuration is usually unique. Therefore, you will likely not be able to merge config changes from one environment branch to another easily.

**Pros**
- Single source 

**Cons**
- Config Drift between environments
- Config promotion difficult
- No separation of concern

## Separate Repos
Storing application code and configuration files in separate repos is the recommended approach. This strategy solves two major issues: configuration drift and lifecycle management.

**Pros**
- No config drift between environments
- Config change promotion simplified
- Separation of concern

**Cons**
- Multiple repos

