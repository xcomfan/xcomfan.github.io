---
layout: page
title: "Undoing/reverting a merge"
permalink: /gitref/branching/undoing_merge
---

[comment]: <> (TODO: REV MARKER)

Lets say you started work on a topic branch, accidentally merged it to master and want to undo the error.

There are two ways to approach this problem.  

## Fix the reference

If the unwanted commit only exists on your local repository, the easiest and best solution is to move the branches so they point to where you want the to.

In most cases you can follow the problematic git merge command with `git reset --hard HEAD~`  This will reset the branch pointers effectively undoing the merge.

The downside of this approach is that you are rewriting history which is problematic for a shared repository.  It also does not work if you made commits after making the mistake.  

## Reverse the commit

Git gives you the option of making a new commit that undoes all the changes from an existing one.  Git calls this option a "revert"

`git revert -m 1 HEAD`

[comment]: <> (TODO: Need to work though this I am not following what the -m 1 is doing)

The -m 1 flag indicates which parent is the "mainline" and should be kept.  When you invoke a merge into HEAD, the new commit has two parents: the first one is HEAD and the second is the tip fo the branch being merged.  Git will get confused here if you try to merge in the topic branch again as all the topic changes are technically in master.  To work around this you need to un-revert the original merge since now you want to bring back the changes that were reverted out.


[comment]: <> (TODO: Need to go back tot he book and really work though this I am very confused command below is definitely not correct)

`git revert ^M`