---
layout: page
title: "Creating And Switching Branches"
permalink: /gitref/creating_and_switching_branches/
---

[comment]: <> (TODO: REV MARKER)

## Listing local branches

`git branch` The * symbol in the output indicates the branch you currently have checked out.

`git brach -v` List local branches and show the last commit message on each branch.

#### Filter for merged or non merged commits in branch listing.  

`git branch --merged` or `git branch --merged [commit]` for a specific commit.

`git branch --no-merged` or `git branch --no-merged [commit]` for a specific commit.

[comment]: <> (TODO: What is the use case for the above commands?  Need to experiment and figure out)

For example to see what has not been merged into the master branch you would use the command

`git branch --no-merged master`

## Creating a new branch

#### Create new branch and switch to it
 
`git checkout -b <my_new_branch_name>` for example `git checkout -b my_new_feature`

if using Git version 2.23 or never you can also use the switch command.

`git switch -c <new_branch_name>` -c stands for create and you can also use the --create flag

#### Create new branch without switching to the new branch

`git branch <new_branch_name>` for example `git branch my_new_feature`

## Switching between branches

To switch branches it is a good practice to have a clean working tree.  You may need to [stash]({% link gitref/stashing.md %}) your work in order to switch to a different branch.
 
`git checkout <existing_branch_name>` or `git switch <branch_name>`

When switching branches the content of your working directory will be updated to reflect the state of the last commit to the branch you are switching to.

If Git cannot cleanly get the files in your working directory to have the content of the branch you are switching to, it won't let you switch.

#### Return to your previously checked out branch 

`git switch -`

# Renaming a branch

***Note:*** Do not rename branches that are still in use by other collaborators.

Rename a branch locally

`git branch --moved bad_branch_name corrected_branch_name`

To let others see the corrected branch on the remote, push it.

`git push --set-upstream origin corrected-branch-name`

Delete the old branch (you can run the `git branch --all` command to see that old branch still exists)

`git push origin --delete bad_branch_name`