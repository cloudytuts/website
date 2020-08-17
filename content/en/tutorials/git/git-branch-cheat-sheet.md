---
title: "Git Branch Cheat Sheet"
date: 2020-08-17T11:54:05-04:00
draft: false
author: serainville
tags:
  - git
description: |
    Use this cheat sheet to learn how to manage your Git branches and never again forget how to accomplish some of the most common tasks.
---

Git is powerful version control system and it can be difficult keeping track of all the common commands. In this cheat sheet we will give you a break down off the most popular, most used commands for you to quickly reference as needed.

## Branches
Git branches are used to organize code changes to our repository, keeping them well away from our release branches until the development work is complete. As mentioned above, Git is a powerful version control system, which working on something as mundane as branches can attest to. 

The following are the most common actions you will typically use against branches when working with Git.

### Switch Branches
Switching to different branches is common practice when working with version control systems. Git has two commands that can be used for switching branches. The first is the `git switch` command.

```shell
git switch <branch-name>
```


### Checkout Branch
An alternative to the `git switch` command for changing branches is the `git checkout` command. 

```shell
git checkout <branch-name>
```

### Checkout New Branch
To create a new branch and check it out you use the `git checkout` command with the `-b` flag.

```shell
git checkout -b <new-branch>
```

### New Branch
To create a new branch you use the `git branch` command and specify the name of the new branch to be created. The branch will be created based on the current commit.

```shell
git branch <new-branch>
```

For example, to create a branch named `feature/add-user-profile` you would run the git command as demonstrated below.

```shell
git branch feature/add-user-profile
```

### New Branch based on CommitID
When creating a new branch you can base it off of an existing commit ID. To do that you use the following `git` coommand syntax.
```shell
git branch <branch-name> <commit-id>
```
For example, to create a new branch named `feature/add-user-profile` based on a commit ID `1f45f18f` you would execute the command as follows.

```shell
git branch feature/add-user-profile 1f45f18f
```

### New Branch based on Remote
When you need to work on an existing remote branch you can pull down with the `git branch` command with the `--track` flag. The syntax of pulling a remote branch is as follows:

```shell
git branch --track <new-local-branch> origin/<remote-base-branch>
```

For example, to pull down a remote branched named `fix/invalid-form-submission` you would execute the following git command.

```shell
git branch --track fix/invalid-form-submission origin/fix/invalid-form-submission
```

### Delete Local Branches
Local branches should be removed from your local repository when they have fullfilled this purpose. To remove a branch from your local repository use the `git branch` command with the `-D` flag.

```shell
git branch -D <branch-name>
```

This operation will only remove the branch from your local repository. If there is a matching remote branch it will be left alone.

### Delete Remote Branches
Just as a local repository should be kept clean and organized by removing uneeded branches, so should the remote repository. In order to delete branches from a remote repository you must perform a `git push` with the `--delete` flag.

```shell
git push --delete origin/<remote-branch-name>
```

### Push Branch to Remote
Branches can be pushed up to a remote branch to be shared with others. This is common practice with branching strategies such as gitflow, where new features are coded in a local feature branch and then pushed up to the remote branch for further review, where it is then merged into master after being review in a pull request.

```shell
git push origin <branch-name>
```