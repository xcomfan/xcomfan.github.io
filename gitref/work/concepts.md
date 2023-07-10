---
layout: page
title: "Concerpts For Working Locally"
permalink: /gitref/work_locally/concepts
---

## The three trees of Git

 A Git project has three trees (tree here means a collection of files not the data structure) that it manages.  

### The working directory (working tree)

The working directory (also called the working tree) is a single checkout of one version of the project.  You can think of the working directory as a sandbox where you can try changes before promoting them to the staging area (index) and eventually committing them.  Internally Git stores files in a compressed format, but for the working directory it extracts them onto the disk for you and makes them available in the working directory so that they can be easily modified.

### The staging area (index)

The index/staging area is where you place your changes in order to commit them.  When you use the Git commit command, Git converts the changes that you staged (place in staging area) into a commit.

[comment]: <> (TODO: Good idea to take some of the content from the internals section and cover exactly what a commit is here.)

### The HEAD

The HEAD "tree" is a pointer to the current branch reference, which in turn is a pointer to the last commit on that branch.  You can think of HEAD as the snapshot of your last commit on the branch you are currently working on.

## The 3 states of your files in Git

As you are working locally in your Git project, the files in your working directory can be in one of the 3 states below.  These 3 states correspond to one of the three trees of Git above.

| State | Description | Corresponding Tree |
| ----- | ----------- | ------------------ |
| modified | You have changed the file, but have not yet committed it to the database | working tree |
| staged | You have marked a file in its current version to go into your next commit snapshot | index |
| committed | The file state is safely stored in your Git database | HEAD |

## Git local workflow

{% mermaid %}

flowchart TD

modify([Modify files])
undo_mod([restore modified file to last committed state])
stage([Stage changes for next commit])
unstage([Un-stage a file])
commit([Commit files in the staging area])
undo_committ([Undo commit and keep changes])
reset_hard([Undo commit and discard changes])

modify-->stage
modify-.optional.->undo_mod
undo_mod-->modify
stage-->commit
stage-.optional.->unstage
unstage-->modify
commit-.optional.->undo_committ
undo_committ-->modify
commit-.optional.->reset_hard
reset_hard-->modify

{% endmermaid %}