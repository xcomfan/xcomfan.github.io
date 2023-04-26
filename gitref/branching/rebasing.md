---
layout: page
title: "Branching"
permalink: /gitref/branching/rebasing/
---

[comment]: <> (TODO: Need to have a section here on what can go wrong and how to recover)

## Rebasing

Rebasing is an alternative approach to merging for integrating changes from one branch to another.  Rebasing replays changes from one line of work onto another in the order they were introduced, whereas merging takes the endpoints and merges them together.

### Rebasing explained

In the diagram below there is work that diverged and there are commits on two different branches.

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

Rebasing makes for a cleaner history.  If you look at the log of a rebased branch, it looks like a liner history.  It appears that the work happened in series when it actually happened in parallel.

[comment]: <> (TODO: Double check if you would rebase onto the work or rebase the work onto your work.)
Rebasing is often used to make sure that your commits apply cleanly on a remote branch - perhaps in a project to which you're trying to contribute but that you don't maintain.  In this use case you would work in a branch and then rebase your work onto origin/master when you are ready to submit your patches to the main project.  This way the maintainer does not need to do any integration work.  Just a fast forward for a clean apply

### Other users for Rebase

You can use rebase to apply topic branches that were created from another topic branch onto your mainline.  The diagram below has a client branch that was created from the server branch. (client branch was created off of the server branch)

[comment]: <> (TODO: I seem to be missing a commit 7 here need to go back to the book and validate.)

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

When you rebase you are abandoning existing commits and creating new ones that are similar but different.  If you push commits somewhere and others pull them down and base work on them, and then you rewrite those commits with git rebase and push them again, your collaborators will have to re merge their work an things will get messy when you try to pull their work back into yours.

### Rebase vs. merge

When you merge branches you see the full history of your repository.  ALl the experimental and dead end branches will be visible in the repository history.  Rebasing lets you have a cleaner history.