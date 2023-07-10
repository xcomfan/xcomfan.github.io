---
layout: page
title: "Remote Branches"
permalink: /gitref/branching/remotes
---

[comment]: <> (TODO: REV MARKER)

## Working with remote branches

[comment]: <> (TODO: Maybe just move the below explanation to definitions and just link it here to keep verbiage minimal.)

[comment]: <> (TODO: I don't love this explanation maybe rewrite it in a way that makes sense to me.)

Remote-tracking branches are local references to the state of remote branches.  You cannot move these references; Git moves them for you whenever you do any network communication to make sure they accurately represent the state of the remote repository.  Think of them as bookmarks to remind you where the branches in your remote repositories were the last time you connected to them.

Remote tracking branch names take the form \<remote\>/\<branch\> for example if you wanted to see what the master branch on your remote origin looks like as of the last time you communicated with it you can check out the origin/master branch.

If you are working on an issue with a partner, you can have a local issue123, but the branch on the server would be represented by the remote-tracking branch origin/issue123

When you clone a git repository, Git makes the clone, but also creates a local master for you to work from.

To synchronize your work with a given remote, you run the command

`git fetch <remote>` for example `git fetch origin`

This command looks up which server origin is, fetches any data from it that you don't have yet and updates your local database moving the origin/master pointer to its new more up-to-date position.

## Pushing

When you want to share a branch with the world, you need to push it up to a remote to which you have write access.  You need to explicitly push the branches you want to share.  This lets you control what you want locally and what you want visible to your collaborators or the public.

To push a local fix branch named bugfix use the command

`git push origin bugfix`

If you want the remote name to be different you can use the command

`git push origin bugfix:newremotename`

It is important to note that when you do a fetch that brings down remote tracking branches, you don't automatically have local, editable copies of them.  For example if you want a local copy of the bugfix branch, you would run the command.

`git checkout -b bugfix origin/serverfix`

## Tracking Branches

Tracking branches are local branches that have a direct relationship to a remote branch.  If you are on a tracking branch and you run the command `git pull` Git automatically knows which server to fetch from and which branch to merge in.

When you clone a repository Git automatically creates a local master branch that automatically tracks origin/master.

To see what tracking branches you have set up you can use the -vv option to git branch

`git branch -vv`

Note the information displayed showing how far ahead/behind you are from remotes is based on local data.  To pull the latest use the commands

`git fetch --all; git branch -vv`

You can set up your own tracking branches (ones that track branches on other remotes or don't track the master branch).

To check out a remote branch that is not master and track it use

`git checkout -b <branch> <remote>/<branch>` or `git checkout --track <remote>/<branch>`

As this practice is so common if you run `git checkout <branch>` and your branch name does not exist locally and exactly matches a name on only one remote git will automatically create a tracking branch.

If you already have a local branch and want to set the upstream branch for it use the command

`git branch -u <remote><branch>` for example `git branch -u origin/serverfix`

If you wish to name the local branch a different name than the remote use

`git checkout -b <my_branch_name> <remote><branch>`

## Pulling git git fetch vs git git pull

The command `git fetch` will download all of the changes from the server that you don't have locally yet, but it will not modify your working directory.  It downloads the data and lets you merge it yourself when you decide to.  The command `git pull` is essentially running `git fetch` and immediately calling git merge.

[comment]: <> (TODO: What is the actual command to git merge manually?)

If you have a tracking branch set up `git pull` will lookup what server and branch your current branch is tracking, fetch from that server, and then try to merge in the remote branch.

[comment]: <> (TODO: Try the above workflow.  If you rebase on origin/master doesn't that change the remote?)

# Deleting a remote branch

To delete a remote branch use the command below. 

`git push origin --delete <branchname>`

# Changing the master branch name

***Note:*** Changing a branch such as master/main/mainline/default can break scripts and integrations.  Make sure you know what you are doing.

Rename your local master branch to main with the following command

`git branch --move master main`

To let others see the new main branch, you need to push it to the remote with the command

`git push --set-upstream origin main`

In the current state the master branch is gone and replaced with the main branch.  The main branch is present on the remote, however the old master branch is still present on the remote, and collaborators will continue to use the master as the base of their work until you make the following changes.

* Any project that depends on this one will need to update their code and/or configuration.
* Update any test-runner configuration files
* Adjust build and release scripts
* Redirect settings on your rep host for things like the repo's default merge rules and other things that match branch names
* Update references to the old branch in documentation
* Close and merge any pull requests that target the old branch

After you have done the above tasks and are certain the main branch performs just as the master branch, you can delete the master brach with teh command

`git push origin --delete master`