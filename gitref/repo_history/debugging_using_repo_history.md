---
layout: page
title: "Debugging Using Repository History"
permalink: /gitref/repo_history/debugging_using_repo_history
---

[comment]: <> (TODO: REV MARKER)

## Debugging with Git

### File annotation (git blame)

File annotations is the best tool if you want to track down when something such as a bug was introduced to the code and why.  File annotations will show you what commit was the last to modify each line of any file.  So for example if you see a method in your code is buggy you can annotate it with git blame to determine which commit was responsible for the introduction fo that line.

`git blame -L 69,82 problemfile.txt`

The -L option allows you to specify a range of lines to display the git blame output for.

The resulting output will show.

* partial SHA-1 of the commit in first column.
* committer and commit date followed by line number in second column
* a carat (^) symbol at the beginning of the commit SHA-1 indicates that the content was there from the initial commit.

[comment]: <> (TODO: Experiment with teh bit below to better understand it)

You can use git blame with the -C option to try and figure our where snippets of the code withing it originally came from.  This is useful if there was a code refactor.  Git will not just find where the line are but the history of the original location they came from.

`git blame -C -L 123, 145 somefile.txt`

### Binary Search (git bisect)

The bisect command does a binary search though your commit history to help you identify as quickly as possible which commit introduced an issue.

The workflow is that you use git bisect to start things off then mark the current version as bad.  Then you tell Git the last good commit.  Git will then binary search though the commits in between to help you identify which commit  was the bad one.  

```
git bisect start
git bisect bad
git bisect good v1.0
```

You can keep using `git bisect good` and `git bisect bad` to test the various commits Git rolls you back to in the binary search.

When you are done with your search, you can use `git bisect reset` to reset your HEAD to where it was before you started.

If you have a script that will exist with a 0 if your project is good or non zero if bad you can automate the process.

```
git bisect start HEAD v1.0
git bisect run test-error.sh
```

Doing this automatically runs test-error.sh on each checked out commit until Git finds the first broken commit.