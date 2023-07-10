---
layout: page
title: "Cleaning Your Working Directory"
permalink: /gitref/work_locally/clean
---

The git clean command is useful for removing untracked files from your working directory.  ***Note:*** files removed with this command cannot be retrieved.

To remove any untracked files and also any subdirectories that become empty use the command

`git clean -f -d`

The -f option is the force option and -d will make it recourse.

By default git clean will not operate on files that are in teh .gitingore file.  You can use the -x option to have it remove those files as well.  This is useful for removing build artifacts in a script.