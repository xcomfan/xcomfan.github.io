---
layout: page
title: "Manual File Re-merging"
permalink: /gitref/branching/manual_file_remerge
---

An alternative to conflict resolution markers is to get copies of the 3 states of conflicting files so they can be compared via diff or other means.  When you have a merge conflict in a file there are essentially 3 states of the conflicting file.

* The merge conflict state
* The "my version" of the file (the one you are trying to check in after merge is complete)
* The "their" version (the one you are trying to merge in)

Git gives you access to these files under stages that have a number associated with them.

* Stage **1** The common ancestor

* Stage **2** is your version

* Stage **3** is the MERGE_HEAD (the theirs version)

To access these states you can use the commands below where hello.rb is the file with a merge conflict.

`git show :1:hello.rb > hello.common.rb`

`git show :2:hello.rb > hello.ours.rb`

`git show :3:hello.rb > hello.theirs.rb`

Once you have the 3 stages in your working directory you can modify them as needed and use the git merge-file command to do a 3 way merge on the 3 file states.  Below is an example of this scenario.

```
dos2unix hello.theirs.rb

git merge-file -p hello.ours hello.common.rb hello.theirs.rb > hello.rb
```

[comment]: <> (TODO: Need to try this merge file thing.  Also from notes in a different section you can use the ours of theirs option when doing the file merge (`git merge-file --ours`) need to test it and document)

After you are done fixing the merge you can use git clean to clear our the files you added but no longer need.

`git clean -f`

You an also use git diff to compare what is in your working directory that you're about to commit as the result of the merge to ay of these stages.

```
git diff --ours

git diff --theirs -b

git diff --base -b
```