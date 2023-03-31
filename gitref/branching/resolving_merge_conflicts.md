---
layout: page
title: "Resolving merge conflicts"
permalink: /gitref/branching/resolving_merge_conflicts
---

## Merge conflicts resolution workflow

Merge conflict happen when there are changes in the same part of a file or files in the two branches you are merging.

If you have merge conflicts in your merge, `git status` will show you which files are in conflict.

After you resolve a conflict use `git add` to mark the file as resolved (staging a file marks it as resolved).  You can use `git status` to confirm all conflicts have been resolved.

Once all the merge conflicts are resolved, you can use `git commit` to create the merge commit.  This creates a new commit for the changes made in the conflict resolution.  Git will provide a default commit message, but you have the option to replace that.  

You can also use `git merge --continue` to have Git walk you though the merging workflow and show if you still have merge conflicts to resolve of if you can commit the merge.

### Conflict resolution markers

Git will add conflict resolution markers to the files that have conflicts to help you resolve the conflicts.

Below is an example of the conflict resolution markers Git adds to the conflicting file(s)

```
<<<<<<< HEAD
# Change from branchA
=======
# Change from branchB
>>>>>>> branchB
```
### Manual file re-merging

In certain circumstances it may be more convenient to view the merge conflict as 3 version of the conflicting file (your version, the version from the branch being merged in, and the common ancestor) instead of looking at many conflict resolution markers.  This process is called manual file re-merging and is documented [here]({% link gitref/branching/manual_file_re_merging.md %})

### Checking out conflicts

Git checkout with the --conflict option will re-checkout the file again and replace the merge conflict markers.  This can be useful if you want to reset the conflict markers and try the merge again.

You can pass `--conflict` either `diff3` or `merge`.  With `diff3` you will get a slightly different conflict output showing you the ours, theirs and base version. 

`git checkout --conflict merge myfile.txt`

`git checkout --conflict diff3 myfile.txt`

[comment]: <> (TODO: Need to experiment with this and move it to configuration section)

If you prefer the diff3 version of the output you can make that your default with the command.

`git config --global merge.conflictstyle diff3`

The diff3 option can be particularly useful when merging large binary files where you just need to pick one side or when you are merging large files from other branches.  You can do the merge then checkout certain files from one side or the other before committing.

### Viewing commit details from branches being merged.

[comment]: <> (TODO: Once you have a write up for the triple dot syntax link to that so its not some random concept you are tossing out there)

You can use the triple dot syntax to get a full list of the unique commits that were included in either branch involved in this merge.

[comment]: <> (TODO: Need an explainer on what MERGE_HEAD is)

`git log --oneline --left-right HEAD...MERGE_HEAD`

This gives you a list of the commits involved, as well as which line of development each commit was on.

The `--merge` option to git log will only show the commits in either side of the merge that touch a file that's currently conflicted.

`git log --oneline --left-right --merge`

If you run that with the `-p` option you get just the diffs to the file that ended up in conflict.  This can be really helpful in quickly giving you the context you ned to help understand why something conflicts and how to more intelligently resolve it.