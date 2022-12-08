---
layout: page
title: "Working on your code"
permalink: /gitref/work/
---

## The 3 states of your files in Git

As you are working locally the files in your working directory can be in one of the 3 states below.

| State | Description |
| ----- | ----------- |
| modified | You have changed the file, but have not yet committed it to the database |
| staged | You have marked a file in its current version to go into your next commit snapshot |
| committed | The file state is safely stored in your Git database |

## Viewing the status of your working tree

`git status`

This command will display the following details

| Section | Purpose |
| ------- | ------- |
| "On branch" | Branch you are currently on TODO: Link to Branching section once you have it.  Probably need to rephrase "working on"|
| "No commits yet" | Only displayed if repository has no commits/history |
| "Changes to be committed" | The staged changes the will be committed when you run the `git commit` command |
| "Changes not staged for commit" | Modified files in your working copy that are not staged to be committed |
| "Untracked Files" | Files in your working copy that Git is not tracking |

## Recording Changes In Your Working Tree

### Stage new files or changes to a tracked files

Add a single file

`git add somefile.txt`

Add all modified and new files

`git add .`

You can add with globbing 

`git add *.py`

***Note:*** Git stages files exactly as they are at the moment of staging.  This means that if you `git add` a file and then modify it you will see it in both "Changes to be committed" and "Changes not staged for commit" sections when you run the `git status` command.

### Un-stage a file

If you staged a file by mistake you can un-stage it with the command

`git restore --staged myfile.txt`