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

## Searching 

Git ships with a built in grep that allows you to search though any committed tree, the working tree, or even the index for a string of regular expressions.  The benefit of git grep over regular grep is that its faster and you can search through any gree in Git not just the working directory.

By default git grep will look through files in your local directory.  

You can use the -n or --line-number option to print out the line numbers where Git has found matches.

`git grep -n my_magic_function` or `git grep --line-number my_magic_function`

[comment]: <> (TODO: Look at man page/help and write up some common use cases here.)

There are many options to git grep

You can use -c or --count to get the file names and found instance counts.

`git grep -c TODO` or `git grep --count TODO`

[comment]: <> (TODO: Need to test this on some code currently I am not clear on what this does.)

You can have git show you the preceding line of function matches with the -p or --show-function argument

`git grep -p my_func *.c` or `git grep --show-function my_func *.c`

You can search for complex combinations of strings with the --and flag, which ensures that multiple matches must occur in the same line of text. 

The example below will search for lines that define a constant whose name contains either of the substrings LINK or BUF_MAX specifically in an older version of the codebase by specifying the tag v1.8.0

`git grep --break --heading -n -e '#define' --and \( -e LINK -e BUF_MAX \) v1.8.0`

* --break argument will print empty lines between matches from different files

* --heading argument will show the file name above the matches

* -n or --line-number will add line numbers to output

* -e argument specified that the next parameter in command is the pattern.  

