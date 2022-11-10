---
layout: page
title: "Initialization"
permalink: /gitref/init/
---

## Creating brand new repository

`git init my_new_repo`

This command will create a new directory named my_new_repo with the .git subdirectory.

## Cloning a repository

`git clone https://github.com/libgit2/libgit2`

This command creates a directory named libgit2 on your system initializes a .git directory inside and pulls down all the data for that repository.  It will olso check out a [working copy]({% link gitref/definitions.md %})

***Note:*** When you clone a repository you are not just getting the working copy.  You will get a full copy of the repository with all the commit history.

**Cloning into a directory with a different name**

`git clone https://github.com/libgit2/libgit2 mydirname`