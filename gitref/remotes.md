---
layout: page
title: "Working with remote repositories"
permalink: /gitref/remotes/
---

[comment]: <> (TODO: Go over this section and see if you can organize it more as a workflow than just a list of commands)

* The term remote does not necessarily mean that the "remote" repository is elsewhere on a network.  Remote just means elsewhere and can be a different directory on your system.

# Showing your remotes

* Having multiple remotes essentially means you are working with multiple collaborators.

see the shortnames for your remotes

`git remote`

see the shortnames and the url of the remote

`git remote -v`

[comment]: <> (TODO: Explain the fetch and push you see int his display)

# Adding remote repositories

To add a remote repository use the command below.  Once added you can refer to the repository with the short name.

`git remote add <shortname> <url>`

# Fetching and pulling from your remotes

To get data from your remote projects you run the command below.  This command goes to the remote project and pull down all the data from that remote project that you don't have yet.  After you do this, you should have references to all the branches from that remote which you can merge in or inspect at any time.

`git fetch <remote>`

***Note*** git fetch only downloads the data to your local repository.  it does not merge it with any of your work or modify what you are currently working on.  You need to manually merge it into your work when you are ready.

* If your current branch is set up to track a remote branch you can use git pull command to automatically fetch and then merge that remote branch into your current branch.

[comment]: <> (TODO: Make a definitions entry for tracking and maybe write a bit about the workflow of this.)

* By default the `git clone` command automatically sets up your local master branch to track the remote master branch (or whatever the master branch is called) on the server you are cloning from.  

* Running git pull generally fetches data from the server you originally cloned from and automatically tires to merge it into the code yo're currently working on.

# Pushing to your remotes

When your project is at a point that you want to share, you need to push it upstream.  The command for that is

[comment]: <> (TODO: Make a definitions entry for upstream and downstream)

`git push <remote> <branch>` 

for example

`git push origin master`

This command will only work if you have write permissions to the remote repository.

If you and someone else clone at the same time and they push upstream and then you push upstream, your changes will be rejected.  You will need to fetch their work first and incorporate it into yours before you will be allowed to push.  This is covered in branching.

[comment]: <> (TODO: Need to add a link above for branching.)

# Inspecting a remote

To find out hte URL for a remote repository as well as tracking information as well as a list of branches available on the remote you can use the command.

`git remote show <remote>`

This command is useful to seeing which branch you would be pulling from and pushing to.

# Renaming a remotes

To change the shortname of a remote use the command

`git remote rename <old_shortname> <new_shortname>` for example `git remote rename abc xyz`

***Note*** This will change your remote-tracking branch names as well.  What used to be at abc/master will now be at xyz/master

[comment]: <> (TODO: Need to do a bit of experimenting to see where/if the above info is useful)

# Remove a remote

If a server was moved, or are no longer using a particular mirror, or a contributor is no longer contributing and you need to delete a remote use the command.

`git remote remove <shortname>` for example `git remote remove xyz`