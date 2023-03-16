---
layout: page
title: "Submodules"
permalink: /gitref/submodules
---

[comment]: <> (TODO: REV MARKER)

## What are submodules

Submodules allow you to keep a Git repository as a subdirectory of another Git repository.  This lets you clone another repository into your project and keep your commits separate.

## Working with submodules

### Adding a submodule to your project

To add a new new submodules use the command below.  By default submodules will add the sub-project into a directory named the same as the repository (DbConnected in the below example).  If you want to specify your own path or directory add that path at the end of the command.  After running this command, in addition to the new directory you will set a **.gitmodules** file added to your repo.  This file is used by people that clone your repo to figure out where your submodules are.  This file is version controlled just like your .gitignore file.

[comment]: <> (TODO: Try this and figure out what this 16XX thing is all about.)
The new directory added by the submodule add command is a special directory in Git (added as 16000 mode meaning its a directory entry in git if viewed via diff).  Git will not track the changes in that directory unless you cd into it.

`git submodule add https://github.com/chaconinc/DbConnector`

[comment]: <> (TODO: Try making this commit and note anything that is not obvious)
***Note:*** You need to commit the change of adding a submodule as you would with any other change.

### Cloning a project with Submodules

[comment]: <> (TODO: Run though this workflow.  The entire section)

When you clone a project with submodules, by default you get the directories that contain submodules, but none of the files within them yet. You need to run the command `git submodule init` to initialize your local configuration file followed by `git submodule update` to fetch all the data from that project and check out the appropriate commit listed in your super-project.  You can also combine the commands as in example below.

`git submodule update --init`

You can also pass the **--recurse-submodules** to the git clone command.  This will automatically initialize and update each submodules in the repository including nested submodules.

### Working on a project with Submodules

Once you have a copy of a project with submodules in it, you can collaborate with teammates on both the main project and the submodule project.

#### Pulling in upstream changes from the submodule remote

The simplest model of using submodules in a project would be if you were simply consuming a sub-project and wanted to get updates from it from time to time but were not actually modifying anything in your checkout

If you want to check for new work in a submodule, you can go into the directory and run git fetch and git merge the upstream branch to update the local code.

```
cd mys_ubmodule_dir
git fetch
git merge origin/master
```

At this point if you go back to the main project and run `git diff --submodules` you will see that the submodule was updated and get a list of commits that were added to it.  If you don't want to type --submodules every time you can make the global config change below.

```
git config --global diff.submodule log
git diff
```

If you commit changes at this point you will lock in the submodule to have this code when other people update.

If you prefer not to manually update and merge you can run the command below.  This will go into your submodule and fetch and update for you.  It will by default assume that you want to fetch and update the default branch of the remote submodule repository.  

`git submodule update --remote`

 If you want this command to track another branch, you can make the setting in your .gitmodules file if you want everyone to track it.  If you want to just track it locally you can make the setting .git/config 

 The command below will change the branch for everyone.  If you want it to be just for you leave off the -f argument.

 `git config -f .gitmodules submodule.DbConnector.branch stable`

 Git will update all of your submodules by default.  If you want it to just update one specify that submodule after the command.

 `git submodule update --remote`

 If you set the configuration setting status.submodulessummary Git will show you a short summary of changes to your submodules.

 ```
 git config status.submodulesummary 1
 git status
 ```

#### Pulling Upstream changes from the project remote

[comment]: <> (TODO: Need to work though the commands in this section to make sense of them and confirm they are correct)

By default `git pull` command recursively fetches submodules changes however it does not update the submodules.  To finalize the update you would need to run git submodules update.  To be ont he safe side you should run submodule update with the --init flag in case the main project commits you just pulled added new submodules, and with the --recursive flag if any submodules have nested submodules.

`git submodule update --init --recursive`

If you want to automate this process, you can add the --recursive-submodules flag to the git pull command.  This will make Git run git submodule update right after the pull, putting the submodules in the correct state.

If you want Git to always pull with --recurse-submodules, you can set the configuration option submodule.recurse to true.

If the upstream main project changed the URL of the submodules you will need to use the command `git submodule sync`

#### Working on a submodule

[comment]: <> (TODO: Need to work though this section and do the formatting as well.)

* In this section we cover making changes to the submodules at the same time as the main project and committing and publishing those changes at the same time.
* If you want your changes to submodules to be tracked you need to take some extra steps.
* If you make a local change to a submodules and run git submodule update --remote --rebase or git submodule update --remote --merge everything should be fine.
* If you forget the --remote and --merge then Git will reset your submodule directory to a detached state.
* If this happens you can still checkout your branch again to get back your changes.
* If you have not committed your changes and run a submodule update that will cause issues.  Git will fetch the changes, but not overwrite the unsaved work in your directory.
* If you made changes with something that conflicts with the upstream Git will tell you when you run the update.

#### Publishing submodule changes

[comment]: <> (TODO: Go back tot the book and complete this section once you work though an example)