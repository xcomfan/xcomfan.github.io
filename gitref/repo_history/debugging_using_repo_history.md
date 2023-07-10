---
layout: page
title: "Debugging Using Repository History"
permalink: /gitref/repo_history/debugging_using_repo_history
---

## Binary Search (git bisect)

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