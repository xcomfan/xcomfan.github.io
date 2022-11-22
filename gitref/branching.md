---
layout: page
title: "Branching"
permalink: /gitref/branching/
---

# What is a Git branch?

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

# Basic branching and merging # 