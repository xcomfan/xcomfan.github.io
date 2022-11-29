---
layout: page
title: "Branching"
permalink: /gitref/branching/
---

[comment]: <> (TODO: right now this is just your notes streamlined need to make this a flow)

# What is a Git branch? #

To understand branching it is helpful to have an idea of how Git stores data.

When you make a commit, Git stores a commit object that contains

* a pointer to a snapshot of the content you staged
* author's name and email address
* the commit message
* pointers to the commit or commits that directly came before this commit (its parents)
    * zero parents for the initial commit
    * one parent for normal commit
    * multiple parents for a commit that results from a merge of two or more branches.

**A branch in git is a lightweight movable pointer to one of these commit objects.**

[comment]: <> (TODO: If the internals section gives you more info on what a snapshot is document that in definitions and link.)

# Creating a new branch #

To create a new branch use the command 

`git branch <new_branch_name>`

This command creates a pointer to the same commit  you are currently on.  Git knows which branch you are currently on because it keeps a special pointer called HEAD that points to the branch you are currently on.  You can see where HEAD is pointing to in the `git log` command output.

***Note:*** The above command just creates a new pointer it does not move where the HEAD pointing to.  In other words it creates the branch, but does not switch you to the newly created branch.

[comment]: <> (TODO: Need confirm this HEAD statement and probably have more content about HEAD someplace once you better understand it.)

# Switching branches #

To switch to an existing branch use the command.  Same command can be used to switch back to the branch you started at.  When switching branches the content of your working directory will be updated to reflect the state of the last commit to the branch you are switching to.

If Git cannot cleanly get the files in your working directory to have the content of the branch you are switching to, it won't let you switch.

`git checkout <existing_branch_name>`

If you want to create a new branch and switch to it in a single command use 

`git checkout -b <my_new_branch_name>`

If you are using Git version 2.23 or newer you can also use the git switch command.

To switch to an existing branch

`git switch <branch_name>`

To create a new branch and switch to it

`git switch -c <new_branch_name>` -c stands for create and you can also use the --create flag

To return to your previously checked out branch 

`git switch -`

# Listing git branches

To list all current branches

`git branch` The * symbol indicates the branch you currently have checked out.

To see the last commit message on each branch

`git brach -v`

Filter for merged or non merged commits.  Note that non merged means branches that have not been merged into the named commit.  If named commit is missing HEAD is assumed.

`git branch --merged` or `git branch --merged [commit]` for a specific commit.

`git branch --no-merged` or `git branch --no-merged [commit]` for a specific commit.

For example to see what has not been merged into the master branch you would use the command

`git branch --no-merged master`

# Basic branching and merging

To switch branches it is a good practice to have a clean working tree.  

[comment]: <> (TODO: Come back here and go though a stashing workflow)

To merge a branch into the branch you are currently working on use the command

`git merge <branch_to_merge_from>`

If the branches you are merging are not diverged (one branch is just ahead of another) when merging Git can just move pointers around (fast forward).  If however, there was a divergence then Git does a 3 way merge between the two branch tips and the common ancestor of the branches being merged.  When this happens git will create a new commit for the merge knows as a merge commit.

[comment]: <> (TODO: Still need to figure out how to tell the difference between regular and merge commit)

To delete a branch you no longer need (once you have merged it into another branch for example) use the -d option

`git branch -d <branch_to_delete>`

# Basic merge conflicts

Merge conflicts happen when you change the same part of a file in two branches you are trying to merge.

git status will show you which files are in conflict.  Git will also add conflict resolution markers to the files that have conflicts to help you resolve the conflicts.

[comment]: <> (TODO: What are these markers???)

After you resolve a conflict use `git add` to mark the file as resolved (staging a file marks it as resolved).  You can also use `git-status` to confirm all conflicts have been resolved.

Once you are happy with your merge, you can use `git commit` to commit the merge.  Git will provide a default commit message, but you have the option to replace that.

[comment]: <> (TODO: git mergetool can be used to help you resolve merge conflicts but its a bit glossed over.  Need to do a deep dive into it and write it up better)

# Working with remote branches

[comment]: <> (TODO: Maybe just move the below explanation to definitions and just link it here to keep verbiage minimal.)

[comment]: <> (TODO: I don't love this explanation maybe rewrite it in a way that makes sense to me.)

[comment]: <> (TODO: I think I may want to move this content to the remote section.)

Remote-tracking branches are local references to the state of remote branches.  You cannot move these references; Git moves them for you whenever you do any network communication to make sure they accurately represent the ster of the remote repository.  Think of them as bookmarks to remind you where the branches in your remote repositories were the last time you connected to them.

Remote tracking branch names take the form \<remote\>/\<branch\> for example if you wanted to see what the master branch on your remote origin looks like as of the last time you communicated with it you can check out the origin/master branch.

If you are working on an issue with a partner, you can have a local issue123, but the branch on the server would be represented by the remote-tracking branch origin/issue123

When you clone a git repository, Git makes the clone, but also creates a local master for you to work from.

To synchronize your work with a given remote, you run the command

`git fetch <remote>` for example `git fetch origin`

This command looks up which server origin is, fetches any data from it that you don't have yet and updates your local database moving the origin/master pointer to its new more up-to-date position.



# Renaming a branch

***Note:*** Do not rename branches that are still in use by other collaborators.

Rename a branch locally

`git branch --moved bad_branch_name corrected_branch_name`

To let others see the corrected branch on the remote, push it.

`git push --set-upstream origin corrected-branch-name`

Delete the old branch (you can run the `git branch --all` command to see that old branch still exists)

`git push origin --delete bad_branch_name`

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