---
layout: page
title: "Branching"
permalink: /gitref/branching/
---

[comment]: <> (TODO: right now this is just your notes streamlined need to make this a flow)

## What is a Git branch?

To understand branching it is helpful to have an idea of how Git stores data.

When you make a commit, Git stores a commit object that contains

* a pointer to a snapshot of the content you staged
* author's name and email address
* the commit message
* pointers to the commit or commits that directly came before this commit (its parents)
    * zero parents for the initial commit
    * one parent for normal commit
    * multiple parents for a commit that results from a merge of two or more branches.

**A branch in git is a lightweight movable pointer to one of these commit objects.**

[comment]: <> (TODO: If the internals section gives you more info on what a snapshot is document that in definitions and link.)

## Creating a new branch

To create a new branch use the command 

`git branch <new_branch_name>`

This command creates a pointer to the same commit  you are currently on.  Git knows which branch you are currently on because it keeps a special pointer called HEAD that points to the branch you are currently on.  You can see where HEAD is pointing to in the `git log` command output.

***Note:*** The above command just creates a new pointer it does not move where the HEAD pointing to.  In other words it creates the branch, but does not switch you to the newly created branch.

[comment]: <> (TODO: Need confirm this HEAD statement and probably have more content about HEAD someplace once you better understand it.)

## Switching branches

To switch to an existing branch use the command.  Same command can be used to switch back to the branch you started at.  When switching branches the content of your working directory will be updated to reflect the state of the last commit to the branch you are switching to.

If Git cannot cleanly get the files in your working directory to have the content of the branch you are switching to, it won't let you switch.

`git checkout <existing_branch_name>`

If you want to create a new branch and switch to it in a single command use 

`git checkout -b <my_new_branch_name>`

If you are using Git version 2.23 or newer you can also use the git switch command.

To switch to an existing branch

`git switch <branch_name>`

To create a new branch and switch to it

`git switch -c <new_branch_name>` -c stands for create and you can also use the --create flag

To return to your previously checked out branch 

`git switch -`

## Listing local branches

To list all current branches

`git branch` The * symbol indicates the branch you currently have checked out.

To see the last commit message on each branch

`git brach -v`

Filter for merged or non merged commits.  Note that non merged means branches that have not been merged into the named commit.  If named commit is missing HEAD is assumed.

`git branch --merged` or `git branch --merged [commit]` for a specific commit.

`git branch --no-merged` or `git branch --no-merged [commit]` for a specific commit.

For example to see what has not been merged into the master branch you would use the command

`git branch --no-merged master`

## Basic branching and merging

To switch branches it is a good practice to have a clean working tree.  

[comment]: <> (TODO: Come back here and go though a stashing workflow)

To merge a branch into the branch you are currently working on use the command

`git merge <branch_to_merge_from>`

If the branches you are merging are not diverged (one branch is just ahead of another) when merging Git can just move pointers around (fast forward).  If however, there was a divergence then Git does a 3 way merge between the two branch tips and the common ancestor of the branches being merged.  When this happens git will create a new commit for the merge knows as a merge commit.

[comment]: <> (TODO: Still need to figure out how to tell the difference between regular and merge commit)

To delete a branch you no longer need (once you have merged it into another branch for example) use the -d option

`git branch -d <branch_to_delete>`

## Basic merge conflicts

Merge conflicts happen when you change the same part of a file in two branches you are trying to merge.

git status will show you which files are in conflict.  Git will also add conflict resolution markers to the files that have conflicts to help you resolve the conflicts.

[comment]: <> (TODO: What are these markers???)

After you resolve a conflict use `git add` to mark the file as resolved (staging a file marks it as resolved).  You can also use `git-status` to confirm all conflicts have been resolved.

Once you are happy with your merge, you can use `git commit` to commit the merge.  Git will provide a default commit message, but you have the option to replace that.

[comment]: <> (TODO: git mergetool can be used to help you resolve merge conflicts but its a bit glossed over.  Need to do a deep dive into it and write it up better)

## Working with remote branches

[comment]: <> (TODO: Maybe just move the below explanation to definitions and just link it here to keep verbiage minimal.)

[comment]: <> (TODO: I don't love this explanation maybe rewrite it in a way that makes sense to me.)

[comment]: <> (TODO: I think I may want to move this content to the remote section.)

Remote-tracking branches are local references to the state of remote branches.  You cannot move these references; Git moves them for you whenever you do any network communication to make sure they accurately represent the state of the remote repository.  Think of them as bookmarks to remind you where the branches in your remote repositories were the last time you connected to them.

Remote tracking branch names take the form \<remote\>/\<branch\> for example if you wanted to see what the master branch on your remote origin looks like as of the last time you communicated with it you can check out the origin/master branch.

If you are working on an issue with a partner, you can have a local issue123, but the branch on the server would be represented by the remote-tracking branch origin/issue123

When you clone a git repository, Git makes the clone, but also creates a local master for you to work from.

To synchronize your work with a given remote, you run the command

`git fetch <remote>` for example `git fetch origin`

This command looks up which server origin is, fetches any data from it that you don't have yet and updates your local database moving the origin/master pointer to its new more up-to-date position.

## Pushing

When you want to share a branch with the world, you need to push it up to a remote to which you have write access.  You need to explicitly push the branches you want to share.  This lets you control what you want locally and what you want visible to your collaborators or the public.

To push a local fix branch names bugfix use the command

`git push origin bugfix`

If you want the remote name to be different you can use the command

`git push origin bugfix:newremotename`

It is important to note that when you do a fetch that brings down remote tracking branches, you don't automatically have local, editable copies of them.  For example if you want a local copy of the bugfix branch, you would run the command.

`git checkout -b bugfix origin/serverfix`

## Tracking Branches

Tracking branches are local branches that have a direct relationship to a remote branch.  If you are on a tracking branch and you run the command `git pull` Git automatically knows which server to fetch from and which branch to merge in.

When you clone a repository Git automatically creates a local master branch that automatically tracks origin/master.

To see what tracking branches you have set up you can use the -vv option to git branch

`git branch -vv`

Note the information displayed showing how far ahead/behind you are from remotes is based on local data.  To pull the latest use the commands

`git fetch --all; git branch -vv`

You can set up your own tracking branches (ones that track branches on other remotes or don't track the master branch).

To check out a remote branch that is not master and track it use

`git checkout -b <branch> <remote>/<branch>` or `git checkout --track <remote>/<branch>`

As this practice is so common if you run `git checkout <branch>` and your branch name does not exist locally and exactly matches a name on only one remote git will automatically create a tracking branch.

If you already have a local branch and want to set the upstream branch for it use the command

`git branch -u <remote><branch>` for example `git branch -u origin/serverfix`

If you wish to name the local branch a different name than the remote use

`git checkout -b <my_branch_name> <remote><branch>`

## Pulling git git fetch vs git git pull

The command `git fetch` will download all of the changes from the server that you don't have locally yet, but it will not modify your working directory.  It downloads the data and lets you merge it yourself when you decide to.  The command `git pull` is essentially running `git fetch` and immediately calling git merge.

If you have a tracking branch set up `git pull` will lookup what server and branch your current branch is tracking, fetch from that server, and then try to merge in the remote branch.

[comment]: <> (TODO: Try the above workflow.  If you rebase on origin/master doesn't that change the remote?)

## Rebasing

Rebasing is an alternative approach to merging for integrating changes from one branch to another.  Rebasing replays changes from one line of work onto another in the order they were introduced, whereas merging takes the endpoints merges them together.

### Rebasing explained

In the diagram below there is worked that diverged and there are commits on tow different branches.

{% mermaid %}
 flowchart RL

    C3([C3])-->C2([C2])-->C1([C1])-->C0([C0])
    C4([C4])-->C2
    experiment[experiment]-.-C4
    master[master]-.-C3

{% endmermaid %}

The easiest way to integrate the branches is to merge them.  A merge will perform a three way merge between the two latest branches C4 and C3 and their most common ancestor C2, creating a new snapshot as in the diagram below.

{% mermaid %}
 flowchart RL

    C5([C5])-->C3([C3])-->C2([C2])-->C1([C1])-->C0([C0])
    C4([C4])-->C2
    C5-->C4
    experiment[experiment]-.-C4
    master[master]-.-C5

{% endmermaid %}

Rebasing lets you do this integration of changes without creating the merge commit C5.  Rebasing will take the patch of the change that was introduced in C4 and reapply it on top of C3.

Rebasing lets you take all of the changes that were committed on one branch and replay them on a different branch.  In the above example above you would rebase with the commands below.  This sequence of commands will check out the experiment branch and then rebase it onto the master branch.

This operation works by going to the common ancestor of the two branches (the one you are on and the one you are rebasing onto), getting the diff introduced by each commit  of the branch you are on, saving those diffs to temporary files, resetting the current branch to the same commit as the branch you are rebasing onto, and finally applying each change in turn.

`git checkout experiment`

`git rebase master`

`git checkout master`

`git merge experiment` This merge will just be a fat forward merge as the changes are already integrated.

At this point your history will look like as the diagram below.  Note that the commit C4 is exactly the same as the C5 commit from the resulting history when we used merge.

{% mermaid %}
 flowchart RL

    C4([C4])-->C3([C3])-->C2([C2])-->C1([C1])-->C0([C0])
    experiment[experiment]-.-C4
    master[master]-.-C4

{% endmermaid %}

[comment]: <> (TODO: Work though the above example to make sure that all 4 commands are needed and to get a feel for how this works)
[comment]: <> (TODO: Write a definition entry for fast forward merge)

### Why use reabasing?

Rebasing makes for a cleaner history.  If you look at the log of a rebased branch, it looks like a liner history.  It looks like a linear history.  It appears that the work happened in series when it actually happened in parallel.

Rebasing is often used to make sure that your commits apply cleanly on a remote branch - perhaps in a project to which you're trying to contribute but that you don't maintain.  In this use case you would work in a branch and then rebase your work onto origin/master when you are ready to submit your patches to the main project.  This way the maintainer does not need to do any integration work.  Just a fast forward for a clean apply

### Other users for Rebase

You can use rebase to apply topic branches that were created from another topic branch onto your mainline.  The diagram below has a client branch that was created from the server branch. (client branch was created off of the server branch)


{% mermaid %}
 flowchart RL

    C6([C6])-->C5([C5])-->C2([C2])-->C1([C1])
    C10([C10])-->C4([C4])-->C3([C3])
    C9([C9])-->C8([C8])
    C3-->C2
    C8-->C3
    server[server]-.-C10
    master[master]-.-C6
    client[client]-.-C9

{% endmermaid %}

If you wish to apply to master the changes you made on the client branch, but you are not ready to integrate the changes from server.  You can do this with the command.

`git rebase --onto master server client`

This command basically says "Take the client branch, figure out the patches since it diverged from the server branch, and replay these patches in the client branch as it was based directly off the master branch instead.  After this command your history tree will look like the below.


{% mermaid %}
 flowchart RL

    C9([C9])-->C8([C8])-->C6([C6])-->C5([C5])-->C2([C2])-->C1([C1])
    C10([C10])-->C4([C4])-->C3([C3])
    C3-->C2
    master[master]-.-C6
    client[client]-.-C9
    server[server]-.-C10

{% endmermaid %}

At this point you just need to fast forward master with the commands

`git checkout master`

`git merge client`

At this point your history looks like

{% mermaid %}
 flowchart RL

    C9([C9])-->C8([C8])-->C6([C6])-->C5([C5])-->C2([C2])-->C1([C1])
    C10([C10])-->C4([C4])-->C3([C3])
    C3-->C2
    master[master]-.-C9
    client[client]-.-C9
    server[server]-.-C10

{% endmermaid %}


At this point you can pull in your server branches by rebasing the server branch onto the master branch without having to check it out first by running the command.

`git rebase <basebranch> <topicbranch>`  in this case `git rebase master server`

This will replay your server work on top of your master work.  And once the replay is complete your history will be in the state below so you just need to merge in the serer branch to fast forward the master branch.

`git checkout master`

`git merge server`

At this point its safe to remove the client and server branches since their changes have been integrated and they are no longer needed.

### Perils of rebasing

***Do not rebase commits that exist outside of your repository and that people may have based their work on***

When you rebase you are abandoning existing commits and creating new ones that are similar but different.  If you push commits somewhere and others pull them down and base work on them, and then you rewrite those commits with git rebase and push them again, your collaborators will have to re merge theri work an things will get messy when you try to pull their work back into yours.

### Rebase vs. merge

When you merge branches you see the full history of your repository.  ALl the experimental and dead end branches will be visible in the repository history.  Rebasing lets you have a cleaner history.

# Deleting a remote branch

To delete a remote branch use the command below. 

`git push origin --delete <branchname>`

# Renaming a branch

***Note:*** Do not rename branches that are still in use by other collaborators.

Rename a branch locally

`git branch --moved bad_branch_name corrected_branch_name`

To let others see the corrected branch on the remote, push it.

`git push --set-upstream origin corrected-branch-name`

Delete the old branch (you can run the `git branch --all` command to see that old branch still exists)

`git push origin --delete bad_branch_name`

# Changing the master branch name

***Note:*** Changing a branch such as master/main/mainline/default can break scripts and integrations.  Make sure you know what you are doing.

Rename your local master branch to main with the following command

`git branch --move master main`

To let others see the new main branch, you need to push it to the remote with the command

`git push --set-upstream origin main`

In the current state the master branch is gone and replaced with the main branch.  The main branch is present on the remote, however the old master branch is still present on the remote, and collaborators will continue to use the master as the base of their work until you make the following changes.

* Any project that depends on this one will need to update their code and/or configuration.
* Update any test-runner configuration files
* Adjust build and release scripts
* Redirect settings on your rep host for things like the repo's default merge rules and other things that match branch names
* Update references to the old branch in documentation
* Close and merge any pull requests that target the old branch

After you have done the above tasks and are certain the main branch performs just as the master branch, you can delete the master brach with teh command

`git push origin --delete master`

## Advanced Merging

[comment]: <> (TODO: This is another section page that I need to move to a better easier to find area)

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