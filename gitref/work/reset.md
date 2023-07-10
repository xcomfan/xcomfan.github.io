---
layout: page
title: "Working with Git reset"
permalink: /gitref/work_locally/reset
---

[comment]: <> (As is this section does not explain where you would use this.  Need to rewrite it with how it used approach that fans out into explanations.)

## Git concepts to help understand git reset command

### Git workflow of the "three trees"

[comment]: <> (A good amount of this content can be folded into the basic concepts for working locally section.)

Git's typical workflow is to record snapshots of your project in successively better states, by manipulating the states of the three [trees]({% link gitref/definitions.md %}#tree)

Lets say we start with a new directory which is not yet a Git repository with a single file in it.  If we run the `git init` command this will create a Git repository with a *HEAD* reference pointing to the unborn master branch.  At this point only the [working directory]({% link gitref/definitions.md %}#working_directory) has any content.

Now we want to commit our single file, so we use the `git add` command.  This will take the content in our *working directory* and copy it to the [*index*]({% link gitref/definitions.md%}#index).

We then run `git commit`, which takes the contents of the *index**and saves it as a permanent snapshot, creates a commit object which points to that snapshot, and updates *master* to point to that commit.  At this point if you run `git status` you will see no change because all three trees are the same.

Switching  branches or cloning goes though a similar process. When you checkout a branch, it changes *HEAD* to point to the new branch ref, populates your *index* with the snapshot of that commit, then copies the contents of the *index* into your *working directory*.

### The role of the reset

When you run the `git reset` command, Git manipulates the three trees doing up to three operations depending on the options specified.

[comment]: <> (TODO: Its not super clear here if you actually run git rest or git reset HEAD~ need experiment)

### Step 1: Move the HEAD (--soft)

The first thing reset will do is move what *HEAD* points to.  This is not the same as changing HEAD itself (which is what checkout does); reset moves the branch that *HEAD* is pointing to.

All calls to git reset have this as the first step.  If you specify the **--soft** argument, git reset will stop here.  This essentially undoes the last commit.  At this stage if you run `git commit`, Git creates a new commit and moves the branch that HEAD points to up to it.  When you reset back to HEAD~ (the parent of HEAD), you are moving the branch back to where it was, without changing the index or working directory (The working directory and index stay the same, but you won't see the previous commit in `git log`).

At this point you can run `git commit` again to accomplish what `git commit --amend` would have done.

### Step 2: Updating the index (--mixed)

Step two is the default for git reset.  If you run git reset without any argument it will stop at this step 2.  In step 2 reset will update the index with the contents of whatever snapshot HEAD points to.  The --mixed argument is optional.

At this step your last commit is still undone, but it will also un-stage everything. In other words `git reset HEAD~` will undo your last commit and add commands.

### Step 3: Updating the working directory (--hard)

The third ting that reset will do (if you specify the --hard) option is to make the working directory look like the index.

In other words git `reset --hard HEAD~` will undo the git commit command your for your prior commit, the git add command for your prior commit, and eliminate any changes you made to the files in your working directory.

### Git reset with a path

You can also provide git reset with a path to act on.

If you specify a path, reset will skip step 1 and limit the remainder of its actions to a specific file or set of files.  This makes sense because HEAD is a pointer and you can't point to a part of a commit, but index and working directory can be partially updated.

[comment]: <> (TODO: book mentions that you can run reset with a SHA-1 or branch. Need to try the branch option)

If you run the command `git reset file.txt` (this command is essentially a short hand for `git reset --mixed file.txt`) git reset will

1. skip moving the branch HEAD points to
2. make the index look like HEAD

In other words it copies file.txt from HEAD to index.  This has the practical effect of unstaging the file.

You don't have to pull the file from HEAD you can specify the commit to pull that version from.  `git reset eb43b4 file.txt`

You can use the --patch option with git reset to un-stage content on a hunk by hunk basis.

## Using git reset for squashing

[comment]: <> (TODO: Link this section to the interactive rebase squashing section and we probably need a mermaid section fro squashing in some workflow.)

Lets say you have a project where...

* The first commit has one file.
* The second commit added a new file and changed the first.
* The third commit changed the first file again.

If the second commit was a work in progress and you want to squash it down you can...

run the command `git reset --soft HEAD~2` to move the HEAD branch back to an older commit (the most recent commit you want to keep)

then simply runt `git commit` again.

This would remove the second commit from your reachable history (the history you would push)

## Reset versus checkout

### Without path specified

Running `git checkout [branch]` is pretty similar to running `git reset --hard [branch]` in that it updates all 3 trees for you to look like `[branch]`.

There are two important differences however.

1. Unlike `reset --hard`, `checkout` is working directory safe.  It will check to make sure its not blowing away files that have changes to them.  It actually tries to do a trivial merge in the working director so that all the files you have not changed will be updated.  `reset hard` on the other hand, will simply replace everything across the board without checking.

2. The way HEAD is updated is different.  `reset` will move the branch that HEAD points to, `checkout` will move HEAD itself to point to another branch.  So for instance if we have master and develop branches which point to different commits, and we are currently on the develop branch (so HEAD points to it).  If we run `git reset master`, develop itself will now point to the same commit that master does.  If we instead run `git checkout master` develop does not move, HEAD itself does.  HEAD will now point to master.  In both cases we are moving HEAD to point to commit A, but how we do so is very different.  `reset` moves the branch HEAD points to, `checkout` moves HEAD itself.

### With path specified

If you run `checkout` with a file path.  Like `reset` this does not move HEAD.  `checkout` will update the index with that file at that commit, but it will also overwrite the file in the working directory.

[comment]: <> (TODO: This is not much of a comparison, just gives you a what checkout does tidbit.  Should move somewhere else when I reorder thins!)

You can use the --patch option with checkout to select hunks you wish to check out.