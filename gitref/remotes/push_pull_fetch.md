---
layout: page
title: "Pushing, Pulling Fetching With Remote Repositories"
permalink: /gitref/remotes/push_pull_fetch
---

[comment]: <> (TODO: Need to make these more command oriented and less wordy with all explainer moving to the basic concepts page)

## Synchronizing your local repository with remote

To get get the latest content from the remote repository use the command below.  This command will update your tracking branches and downlaod any new ones created on the remote repository which are not in your local repository yet.

`git fetch <remote>` for example `git fetch origin`

## Checkout a remote branch

When you do a fetch that brings down remote tracking branches, you don't automatically have local, editable copies of them.  For example if you want a local copy of the bugfix branch, you would run the command.

`git checkout -b <local_branch_name> remote/remote_branch_name` 

for example 

`git checkout -b bugfix origin/serverfix`

## Pushing to your remotes

When your project is at a point that you want to share, you need to push it upstream.  The command for that is

`git push <remote> <branch>`

for example

`git push origin master`

If you want the remote name to be different you can use the command

[comment]: <> (TODO: Need to try this out and make the branch names being used more consistent.)

`git push origin bugfix:newremotename`

This command will only work if you have write permissions to the remote repository.

If you and someone else clone at the same time and they push upstream and then you push upstream, your changes will be rejected.  You will need to fetch their work first and incorporate it into yours before you will be allowed to push.  This is covered in branching section.

[comment]: <> (TODO: Make a link to section where above is addrssed)

# Deleting a remote branch

To delete a remote branch use the command below. 

`git push origin --delete <branchname>`

# Changing the master branch name

***Note:*** *Changing a branch such as master/main/mainline/default can break scripts and integrations. Make sure you know what you are doing.*

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