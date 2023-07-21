---
layout: page
title: "Working With Remote Branches"
permalink: /gitref/remotes/add_remove_view
---

## List your remote tracking branches

To see what tracking branches you have set up you can use the -vv option to git branch

`git branch -vv`

Note the information displayed showing how far ahead/behind you are from remotes is based on local data.  To pull the latest use the commands

`git fetch --all; git branch -vv`

## Set up tracking branches

You can set up your own tracking branches (ones that track branches on other remotes or don't track the master branch).

To check out a remote branch that is not master and track it use the command below.

`git checkout -b <branch> <remote>/<branch>` or `git checkout --track <remote>/<branch>`

As this practice is so common if you run `git checkout <branch>` and your branch name does not exist locally and exactly matches a name on only one remote git will automatically create a tracking branch.

If you wish to name the local branch a different name than the remote use the following command.

`git checkout -b <my_branch_name> <remote><branch>`

### Setting upstream branch

If you already have a local branch and want to set the upstream branch for it use the command

`git branch -u <remote><branch>` for example `git branch -u origin/serverfix`