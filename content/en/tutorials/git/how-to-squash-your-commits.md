---
title: "How to Squash Your Commits and Why Do It"
date: 2020-08-18T10:11:39-04:00
draft: false
author: serainville
tags:
    - git
description: |
    Learn how to squash your commits into a single commit and understand why you should do it. Keep your commit history clean by only keeping important changes in your log using squash and fixup commands
---

Squashing commits is used to combine multiple commits into a single, larger commit. 

In an ideal world code changes would only be done in short lived feature branches. The reality is we often need to share new snippets with others in a feature branch prior to merging into a release branch, or you find yourself forced to push multiple changes to branch while troubleshooting a pipeline issue.

All of these extraneous commits clutter your repository with noise, making it difficult to identity actual features in your code. Squashing allows you to rebase multiple commits into a single, larger commit, with the goal of not polluting your git log. 

## Rebase Squashing Commits
Squashing commits is the process of rebasing your repository. There a few methods to rebase squash your git history.

Rebasing can be done interactively, which is to say with a temporary file loaded into your shell's default text editor. This is by and far the most recommend way of rebasing, as you can visually see exactly what you are attempting to accomplish. Rebasing will occur after the temporary file is saved. If no syntax errors are detected and the rebase completes successfully, the file will be deleted.

The two most common methods of rebasing are:
* Rebase from HEAD to an `x` number of commits behind
* Rebase from a commit ID

To rebase an `x` number of commits from `HEAD`, you would use the following command syntax with git.
```shell
git rebase -i HEAD~[NUMBER OF COMMITS]
```

Alternatively, the second common method is to specify the commit ID you want to rebase from.
```shell
git rebase -i [COMMIT ID]
```

The `-i` flag instructs Git to perform the rebase interactively.

{{< note >}}
Both methods above as essential identical, in that you are rebasing from one commit ID to another. With `git rebase -i HEAD~10`, you are rebasing from `HEADS` commit ID through to the previous 9 commits.
{{< /note >}}

An interactive rebase will open your default text editor with contents similar to the following. All commits selected will be listed with a command (`pick`), its commit ID, and the log text.

```text
pick cac1790 Add ansible v2.10.0 released
pick 524c030 Add using-unvault-to-read
pick 41532d5 Create FUNDING.yml
pick f4ae04d Add delete git branches
pick 602781a Add git-branch-cheat-sheet
pick 2782c19 Fix typos and grammar
pick 02164c7 Remove old files
pick 10468a4 Add kubernetes-v1.18.7-release
pick 9341aa0 add set-default-kubernetes-namespace
pick 1dc377e Add how-to-deploy-lamp-on-google-cloud-compute

# Rebase ffbe8cb..1dc377e onto 10468a4 (10 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
```

The two primary commands you will use when squashing your commits are `squash` or `fixup`. The difference between these two is the former retains your log message, while the latter discards the log message. 

### Squash last 10 from head
Your `head` is the current position. To squash the ten most recent commits from head into a single commit, for example, you would rebase with `HEAD~10`. 

```shell
git rebase -i HEAD~10
```

To squash commits replace command `pick` in front of each commit your want squashed with `squash`. 

```text
squash cac1790 CLDY-1000: Finally!
squash 524c030 CLDY-1000: Nope, not yet. More bugs.
squash 41532d5 CLDY-1000: I think that's it!
squash f4ae04d CLDY-1000: Bug fix again
squash 602781a CLDY-1000: Bug fix
squash 2782c19 CLDY-1000: Bad code. Who approved this?!
squash 02164c7 CLDY-1000: Fixed text alignment
squash 10468a4 CLDY-1000: More typos...
squash 9341aa0 CLDY-1000: typo fixed
pick 1dc377e CLDY-1000: New user profile
```

When you save your changes and exit the text editor, git will apply the desired commands onto your history. Notice how the first 9 commits have a `squash` command, while the last commit remains as `pick`. This instructs git to squash the first 9 commits into the last commit.


### Squashing from Commit ID
Each commit is logged with a unique identifier, the commit ID. To squash all commits from your current HEAD to a specific commit, we can rebase using a commit ID.

```shell
git rebase -i 10468a4a06659311f55d2573a20437922b24c912
```

## Rebase Fixup Commits
Squashing and fixup are very similar methods of combining commits. The difference is when a fixup is selected the log text for each commit is not retained. This is more useful when you have extraneous commits due to troubleshooting, for example. It's unlikely you will want or need to retain these messages.

```text
fixup cac1790 CLDY-1000: Finally!
fixup 524c030 CLDY-1000: Nope, not yet. More bugs.
fixup 41532d5 CLDY-1000: I think that's it!
fixup f4ae04d CLDY-1000: Bug fix again
fixup 602781a CLDY-1000: Bug fix
fixup 2782c19 CLDY-1000: Bad code. Who approved this?!
fixup 02164c7 CLDY-1000: Fixed text alignment
fixup 10468a4 CLDY-1000: More typos...
fixup 9341aa0 CLDY-1000: typo fixed
pick 1dc377e CLDY-1000: New user profile
```

As with squashes, all commits from `cac1790` to `9341aa0` will be merged into `1dc377e CLDY-1000: New user profile`.