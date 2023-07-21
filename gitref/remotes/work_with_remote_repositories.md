---
layout: page
title: "Working With Remote Repositories"
permalink: /gitref/remotes/add_remove_view
---

## List your remote repositories

see the shortnames for remote repositories you have defined

`git remote`

see the shortnames and the url of the remote repositories

`git remote -v`

[comment]: <> (TODO: Explain the fetch and push you see in this display.  I get what the push and pull do, but how to you set each?)

## Inspect a remote repository

To see the URL for a remote repository as well as tracking information, and a list of branches available on the remote you can use the command.

`git remote show <remote>`

This command is useful to seeing which branch you would be pulling from and pushing to.

## Add a remote repository

To add a remote repository use the command below.  Once added you can refer to the repository with the short name.

`git remote add <shortname> <url>`

## Change shortname for a remote repository

`git remote rename <old_shortname> <new_shortname>` for example `git remote rename abc xyz`

***Note*** This will change your remote-tracking branch names as well.  What used to be at abc/master will now be at xyz/master

[comment]: <> (TODO: Need to do a bit of experimenting to see where/if the above info is useful)

## Remove a remote repository

If a server was moved, or are no longer using a particular mirror, or a contributor is no longer contributing and you need to delete a remote use the command.

`git remote remove <shortname>` for example `git remote remove xyz`