---
layout: page
title: "Starting a merge"
permalink: /gitref/branching/starting_merge
---

Before attempting to merge branches make sure your working tree is clean.  If you have work in progress either [stash]({% link gitref/stashing.md %}) the work or apply your changes to a temporary branch (stash your work and then [apply]({% link gitref/stashing.md %}#Retrieving_your_stashed_work) your stash on a branch).  This makes it easier to undo things if you run into issues resolving merge conflicts.

[comment]: <> (TODO: Beyond stashing and applying your stash to a temporary branch you may be able to use cherry pick for this save to temporary branch.  Come back and update if that seems reasonable?)

The merge command merges a specified branch into the branch you are currently on.  To initiate a merge use the command below.

`git merge <branch_to_merge_from>`

If the branches you are merging are not diverged (one branch is just ahead of another) when merging, Git can just move pointers around (fast forward).  If however, there was a divergence then Git does a 3 way merge between the two branch tips and the common ancestor of the branches being merged.  When this happens git will create a new commit for the merge known as a merge commit.

[comment]: <> (TODO: Still need to figure out how to tell the difference between regular and merge commit)

To delete a branch you no longer need (once you have merged it into another branch for example) use the -d option

`git branch -d <branch_to_delete>`

## Ignore whitespace when merging

A whitespace related conflict looks like every line being being removed on the left side appearing on the right.  By default Git sees all of these lines as being changed, so it can't merge the files.  If you are seeing a lot of whitespace conflicts you can abort your merge and re-attempt it with the command below.  The -Xignore-space-change option ignores whitespace completely when comparing lines.  It will treat sequences of one or more whitespace lines as equivalent.

`git merge -Xignore-space-change <branch_to_merge_from>`

## Ours or theirs preference when merging

[comment]: <> (TODO: I need to actually try the arguments being discussed here to understand how they work.)

When merging you can use the `-Xours` or `-Xtheirs` options to tell Git to to prefer either their or your version.  This means if there is a merge conflict the side you are specifying will win and you don't need to manually resolve merge conflicts.

`git merge -Xours some_branch`