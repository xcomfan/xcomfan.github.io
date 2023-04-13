---
layout: page
title: "Stashing"
permalink: /gitref/stashing/
---

[comment]: <> (TODO: REV MARKER)

Stashing can be used to save your modified tracked files and staged changes.  This saved state can be re applied at any time and to any branch.  

## Viewing your Git stash

`git stash list`

## Stashing your work

`git stash push -m "My optional stash message"`

`git stash push`

Stash your changes but leave what is staged

`git stash push --keep-index`

To save untracked files which are not in your gitignore file along with your other changes in the stash

`git stash push --include-untracked`

## Retrieving your stashed work

To pop from the stash (apply your stashed work and remove it from the stash)

`git stash pop`

To apply your latest stashed work and keep it saved in the stash

`git stash apply`

To apply a specific state of work from your stash use the command below.  The stash versions will be listed in the output of `git stash list`

[comment]: <> (TODO: Make an example of the below command so you know what ti looks like.)

`git stash apply stash@{2}`

When you apply your stash it will not automatically re stage files that were staged when you created the stash.  Use the either of the commands below if you want your files re-staged.

`git stash pop --index`

`git stash apply stash@{2} --index`

## Viewing your stash entries

If you wish to see the difference between your stash content and the commit back when the stash entry was first created you can use the command 

`git stash show -p`  or 

`git stash show -p stash@{1}` if you wish to compare to a specific stash entry not the top one.  

By default this will use the diffstat but you can configure it to use any format known to git diff.

[comment]: <> (TODO: Link above to the config content once you write it.  For how to config it see man git stash page.)

## Removing entries from your stash

To remove a specific entry

`git stash drop stash@{2}`

To completely clear you stash

`git stash clear`

## Create a branch from a stash

The command below will create a new branch and apply your stash to it.  If the stash applies cleanly it will be dropped.

[comment]: <> (TODO: Try out the below.  I am not sure what it does exactly)

`git stash branch <branchname>`