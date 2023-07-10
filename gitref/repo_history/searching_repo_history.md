---
layout: page
title: "Searching Repository History"
permalink: /gitref/repo_history/searching_repo_history
---

## Git log searching

Git log search is usefull when you are looking for when a something got introduced or existed in your Git project instead of where.

If for example you want to find when the ZLIB_BUF_MAX constant was originally introduced, you can use the -S option to git log.  The output of the command will be the commit details of the commit that introduced the variable. If you need to be more specific you can use the -G option to provide a regular expression.

`git log -S ZLIB_BUG_MAX --oneline`

### Line Log Search

you can use Line Log Search to check the history of a function or a line of code.  Just use the -L option with git log.  For example, if we want to see every change made to the function git_deflate_bound in the zlib.c file we would use the command below.  This command will try to figure out what the bounds of that function are and then look though the history and show us every change that was made to the function as a series of patches.

`git log -L :git_deflate_bound:zlib.c`

If Git can't figure out hwo to match a function or method in your programming language, you can also provide it with a regular expression as in the example below.

`git log -L '/unsigned long git_deflate_bound/',/^}/:zlib.c`

## File annotation (git blame)

File annotations is the best tool if you want to track down when something such as a bug was introduced to the code and why.  File annotations will show you what commit was the last to modify each line of any file.  So for example if you see a method in your code is buggy you can annotate it with git blame to determine which commit was responsible for the introduction fo that line.

`git blame -L 69,82 problemfile.txt`

The -L option allows you to specify a range of lines to display the git blame output for.

The resulting output will show.

* partial SHA-1 of the commit in first column.
* committer and commit date followed by line number in second column
* a carat (^) symbol at the beginning of the commit SHA-1 indicates that the content was there from the initial commit.

[comment]: <> (TODO: Experiment with the bit below to better understand it)

You can use git blame with the -C option to try and figure our where snippets of the code within it originally came from.  This is useful if there was a code refactor.  Git will not just find where the lines are but the history of the original location they came from.

`git blame -C -L 123, 145 somefile.txt`