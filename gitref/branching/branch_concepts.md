---
layout: page
title: "Branching Concepts"
permalink: /gitref/branching/concepts
---

[comment]: <> (TODO: REV MARKER)

## What is a Git branch?

To understand branching it is helpful to have an idea of how Git stores data.

When you make a commit, Git stores a commit object that contains the following.

* A pointer to a snapshot of the content you staged
* The commit author's name and email address
* The commit message
* Pointers to the commit or commits that directly came before this commit (the parents)
    * Zero parents for the initial commit
    * One parent for normal commit
    * Multiple parents for a commit that results from a merge of two or more branches.

**A branch in git is a lightweight movable pointer to one of these commit objects.**

## What happens when I run the git branch command.

The command `git branch <new_branch_name>` creates a pointer to the same commit you are currently on.  Git knows which branch you are currently on because it keeps a special pointer called HEAD that points to the branch you are currently on.

[comment]: <> (TODO: This needs more explanation...)

## Merged vs unmerged commits

Note that non merged means branches that have not been merged into the named commit.  If named commit is missing HEAD is assumed.

[comment]: <> (TODO: There is content from the distributed Git chapter that you should add here for now these are just local workflows)

## Branching workflows

### Long running branches

* Only stable code in the master branch.  Possibly just code that has been released.
* There is a parallel branch named develop or next that is used to test stability.
    * This branch is not necessarily always stable, but when it gets to stable state it can be merged into master.
    * This branch is used to pull in [topic branches]({% link gitref/definitions.md %}#topic-branch) and run them though testing.
* Some larger projects have a proposed or pu (proposed updates) branch that has integrated branches which may not be ready to go into the next or master branch yet.
    * The idea is that your branches have various levels of stability.  When they reach a stable level, they are merged into the branch above them.
* The multiple long running branches approach is helpful when you have very large or complex projects.

### Topic Branches

* A topic branch is a short lived branch that you create and use for a single particular feature or related work.
* You create a topic branch for whatever task you are working on and delete it after you merge it into the master branch.
* This technique lets you context switch by separating your work into branches where all work in a branch is for the branch topic.