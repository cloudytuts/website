---
title: "Delete Local and Remote Git Branches"
date: 2020-08-17T10:04:28-04:00
draft: false
author: serainville
tags:
  - git
description: |
    Learn how to delete your Git branches on your local filesystem and from a remote repository, and keep your repository clean from branch clutter.
---

No matter which strategy you follow with version control, whether it be gitflow or some custom branching scheme, you will eventually need to remove branches from your Git repositories. 

## Remove Local Branches
To remove a branch from your local Git repository you the `git branch` command with the `-d` or `-D` flag. 

```shell
git branch -D <branch-name>
```

For example, to delete a branch with the name `feature/user-profile`, you would run the following command:
```shell
git branch -D feature/user-profile
```
If the branch was deleted successfully, the output of the command will show the name of the deleted branch and it's highest commit ID.

```shell
Deleted branch feature/user-profile (was 87beaa4)
```

The `git branch -D` command only applies to local repositories. A different method is required for deleting remote branches from repositories.

## Remove Remote Branches
Fortunately, the `git branch -D` command does not remove a branch from your local repository and any remote repositories. In order to delete a remote repository you must perform a `git push` with the `--delete` flag.

```shell
git push --delete <remote> <branch-name>
```

For example, if your remote repository is named `origin` and the branch you would like to delete is named `feature/password-reset-form`, you will execute the following command.

```shell
git push --delete origin feature/password-reset-form
```
Which will output the following, if the branch is successfully deleted.
```shell
remote: This repository moved. Please use the new location:
remote:   https://github.com/cloudyyuts/demo-app.git
To https://github.com/cloudytuts/demo-app.git
 - [deleted]         feature/password-reset-form
```
