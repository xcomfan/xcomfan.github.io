---
layout: page
title: "Remote Repository Basic Concepts"
permalink: /gitref/remotes/basic_concepts
---

## Remote Repositories

A remote respository is a repository that is used to track the same project which you are working on in your local repository, but the remote repository resides somehwhere else. Git's `fetch` and `push` commands are used to communicate with remote repositories.

The term remote does not necessarily mean that the "remote" repository is elsewhere on a network.  Remote just means elsewhere and can be a different directory on your system.

Having multiple remote repositories essentially means you are working with multiple collaborators.

## Remote Tracking Branches

Remote-tracking branches are references in your local repository to the state of branches in a remote repository.  You annot modify these references directly; Git will update them whenever you do any network communication to make sure they accurately respesent the state of the remote repository.  They are a snaphot of where the branches in a remote repository were the last time you connected to them.

Remote tracking branch names take the form \<remote\>/\<branch\> for example if you wanted to see what the master branch on your remote origin looked like as of the last time you communicated with it you can check out the origin/master branch.

If you are working on an issue with a partner, you can have a local issue123, but the branch on the server would be represented by the remote-tracking branch origin/issue123.

When you clone a Git repository, Git's `clone` command automatically names the remote repository origin, pulls down all the data from the remote repository, creates a pointer to where the remote master branch is and names it `origin/master`.  Git also gives you a local `master` branch starting at the same place as origin's `master` branch so you have something to work from.

## Tracking Branches

Tracking branches are local branches that have a direct relationship to a remote branch.  If you are on a tracking branch and you run the command `git pull` Git automatically knows which server to fetch from and which remote branch to merge into the tracking branch on your local system.  

## Pushing to a remote repository

When you want to share a branch with the world, you need to push it up to a remote repository to which you have write access.  You need to explicitly push the branch you want to share.  This lets you control what you want locally and what you want visible to your collaborators or the public.

## Fetch vs. Pull

The command `git fetch` will download all of the changes from the server that you don't have locally yet, but it will not modify your working directory.  It downloads the data and lets you merge it yourself when you decide to.  Git pull will try to merge the changes from the remote branch in addition to pulling them down from the remote repository.  The command `git pull` is essentially running `git fetch` and immediately calling git merge.

[comment]: <> (TODO: What is the actual command to git merge manually?)

If you have a tracking branch set up `git pull` will lookup what server and branch your current branch is tracking, fetch from that server, and then try to merge in the remote branch.

[comment]: <> (TODO: Try the above workflow.  If you rebase on origin/master doesn't that change the remote?)