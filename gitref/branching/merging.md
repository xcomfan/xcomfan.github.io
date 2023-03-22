---
layout: page
title: "Merging Branches"
permalink: /gitref/branching/merging
---

[comment]: <> (TODO: REV MARKER)

## Basic merging

To merge a branch into the branch you are currently working on use the command

`git merge <branch_to_merge_from>`

If the branches you are merging are not diverged (one branch is just ahead of another) when merging, Git can just move pointers around (fast forward).  If however, there was a divergence then Git does a 3 way merge between the two branch tips and the common ancestor of the branches being merged.  When this happens git will create a new commit for the merge known as a merge commit.

[comment]: <> (TODO: Still need to figure out how to tell the difference between regular and merge commit)

To delete a branch you no longer need (once you have merged it into another branch for example) use the -d option

`git branch -d <branch_to_delete>`

[comment]: <> (TODO: Write upt he -D option you frequently use here)

## Basic merge conflicts

Merge conflicts happen when you change the same part of a file in two branches you are trying to merge.

git status will show you which files are in conflict.  Git will also add conflict resolution markers to the files that have conflicts to help you resolve the conflicts.

[comment]: <> (TODO: Add info on what these markers look like)

After you resolve a conflict use `git add` to mark the file as resolved (staging a file marks it as resolved).  You can also use `git-status` to confirm all conflicts have been resolved.

Once you are happy with your merge, you can use `git commit` to commit the merge.  Git will provide a default commit message, but you have the option to replace that.

[comment]: <> (TODO: git mergetool can be used to help you resolve merge conflicts but its a bit glossed over.  Need to do a deep dive into it and write it up better)

[comment]: <> (TODO: If I mess up a merge how to I get back to the merge conflict state to try again?)


## Advanced Merging

### Merge conflicts

When attempting a merge make sure your working tree is clean.  If you have work in progress either commit it to a temporary branch or stash it.  This makes it so that you can undo things when you try to merge.

If you started a merge and get conflicts you can use the below command to abort the merge.

`git merge --abort`

If for some reason you want to start over you can use `git --reset HEAD` and return your repo to its last committed state. ***Note:*** Any of your uncommitted work will be lost.

### Ignoring whitespace

A whitespace related conflict looks like every line being being removed on the left side appearing on the right.  By default Git sees all of these lines as being changed, so it can't merge the files.  If you are seeing a lot of whitespace conflicts you can abort your merge and re-attempt it with the command below.

[comment]: <> (TODO: Need to try this out to see how it works)

`git merge --Xignore-space-change somebranch`

The --Xignore-space-change option ignores whitespace completely when comparing lines.  It will treat sequences of one or more whitespace lines as equivalent.

### Manual file re-merging

[comment]: <> (TODO: Need to experiment with this and see how it compares to normal take this or that change option)

You will very likely run into situations where Git can't automatically fix things in a merge.  In this scenario git allows you to get copies of the 3 states of the conflicting file.

* One state is the merge conflict state.
* One state is "my version" of the file (the one you are trying to check in after merge)
* The other version is the "their" version (the one you are trying to merge in)

Git stores gives you access to these files under stages that have a number associated with them.

Stage **1** The common ancestor

Stage **2** is your version

Stage **3** is the MERGE_HEAD (the theirs version)

To access these states you can sue the commands below.

`git show :1:hello.rb > hello.common.rb`

`git show :2:hello.rb > hello.ours.rb`

`git show :3:hello.rb > hello.theirs.rb`

Once you have the 3 stages in your working directory you can modify than as needed for example...

[comment]: <> (TODO: Need to dig deeper into the git merge-file command)

```
dos2unix hello.theirs.rb

git merge-file -p hello.ours hello.common.rb hello.theirs.rb > hello.rb
```

[comment]: <> (TODO: Need to experiment with the below as well)

You can use git diff to compare what is in your working directory that you're about to commit as the result of the merge to ay of these stages.

```
git diff --ours

git diff --theirs -b

git diff --base -b
```

After you are done fixing the merge you can use git clean to clear our the files you added but no longer need.

`git clean -f`

### Checking out conflicts

[comment]: <> (TODO: Need to experiment with this and add an actual example command)

Git checkout with the --conflict option will re-checkout the file again and replace the merge conflict markers.  This can be useful if you want to reset the conflict markers and try the merge again.

[comment]: <> (TODO: Need to experiment with this and move it to configuration section)

You can pass --conflict either diff3 or merge.  With diff 3 you will get a slightly different conflict output showing you the ours, theirs and base version. 

`git checkout --conflict merge`

`git checkout --conflict diff3`

If you prefer the diff3 version of the output you can make that your default with the command.

`git config --global merge.conflictstyle diff3`

The diff3 option can be particularly useful when merging large binary files where you just need to pick one side or when you are merging large files from other branches.  You can do the merge then checkout certain files from one side or the other before committing.

### Merge log

[comment]: <> (TODO: Need to work though the examples in this section I am a bit lost at the moment)

You can use the triple dot syntax to get a full list of the unique commits that were included in either branch involved in this merge.

`git log --oneline --left-right HEAD...MERGE_HEAD`

This gives you a list of the commits involved, as well as which line of development each commit was on.

The --merge option to git log will only show the commits in either side of the merge that touch a file that's currently conflicted.

`git log --oneline --left-right --merge`

If you run that with the -p option you get ust the diffs tot he file that ended up in conflict.  This can be really helpful in quickly giving you the context you ned to help understand why something conflicts and how to more intelligently resolve it.

### Combined diff format

Since Git stages any merge results that are successful, when you run git diff while in a conflicted merge state, you only get what is currently still in conflict.  This can be helpful to see what you still have to resolve.  

[comment]: <> (TODO: Combined diff is a good candidate for a definition entry)

When you run git diff directly after a merge conflict it will show you a "Combined Diff".  This format gives you two columns of data nest to each line.  

* The first column shows you if that line is different (added or removed) between the ours branch and teh file in your working directory.
* The second column does the same between the "theirs" branch and your working directory copy.

As you resolve conflicts and run git diff you will see the progress.

If you run git show on a merge commit you will see this format as well.

## Undoing merges

Lets say you started work on a topic branch, accidentally merged it to master and want to undo the error.

There are two ways to approach this problem.  

### Fix the reference

If the unwanted commit only exists on your local repository, the easiest and best solution is to move the branches so they point to where you want the to.

In most cases you can follow the problematic git merge command with `git reset --hard HEAD~`  This will reset the branch pointers effectively undoing the merge.

The downside of this approach is that you are rewriting history which is problematic for a shared repository.  It also does not work if you made commits after making the mistake.  

### Reverse the commit

Git gives you the option of making a new commit that undoes all the changes from an existing one.  Git calls this option a "revert"

`git revert -m 1 HEAD`

[comment]: <> (TODO: Need to work though this I am not following what the -m 1 is doing)

The -m 1 flag indicates which parent is the "mainline" and should be kept.  When you invoke a merge into HEAD, the new commit has two parents: the first one is HEAD and the second is the tip fo the branch being merged.  Git will get confused here if you try to merge in the topic branch again as all the topic changes are technically in master.  To work around this you need to un-revert the original merge since now you want to bring back the changes that were reverted out.


[comment]: <> (TODO: Need to go back tot he book and really work though this I am very confused command below is definitely not correct)

`git revert ^M`

## Ours or theirs preference when merging

[comment]: <> (TODO: This section needs to be moved to be with merging when I reorganize these notes to be smaller chunks.  I also need to actually try the arguments being discussed here to understand how they work.)

When merging you can use the -Xours or -Xtheirs options to git merge to tell Git to to prefer either their or your version.  This means if there is a merge conflict the side you are specifying will win and you don't need to manually do the merging.

`git merge -Xours somebranch`

This option will also work on the merge-file command for individual file  merges.

[comment]: <> (TODO: Need to try this merge-file business)

`git merge-file --ours`