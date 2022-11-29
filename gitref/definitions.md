---
layout: page
title: "Definitions"
permalink: /gitref/definitions/
---

# author

Author in a commit is the person that originally wrote the work vs committer is the person who last applied the work.

# committer

Committer is the person in a commit history entry that applied the work vs the author who is the person that originally wrote the work.

# index

Another term for the Git staging area

# merge commit

A commit created by Git when two branches with diverging changes are merged.  

# snapshot

TODO Need to get a good definition here.

# staging area

A file generally in your Git directory that stores what changes will go into your next commit.  Its technical name in Git is "index"

# topic branch

A topic branch is a short lived branch that you create and use for a single particular feature or related work

# tracked files

File that were in the last snapshot as well as any new staged files.  Tracked files can be either unmodified, modified or staged.

# working copy

When you are working in a Git repository the working copy is the copy on your disk actively being worked on.

# working tree

A single checkout of one version of the project.  These files are extracted from the Git directory and placed on disk for you to use or modify.