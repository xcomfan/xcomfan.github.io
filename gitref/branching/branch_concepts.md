---
layout: page
title: "Branching Concepts"
permalink: /gitref/branching/concepts
---

[comment]: <> (TODO: REV MARKER)

## What is a Git branch?

To understand branching it is helpful to have an idea of how Git stores data.

When you make a commit, Git stores a commit object that contains

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