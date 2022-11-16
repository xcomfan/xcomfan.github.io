---
layout: page
title: "Commiting Your Changes"
permalink: /gitref/commit/
---

# Basic Commit #

Use default editor to write your commit message.

[comment]: <> (TODO: Add link to config section where you specify what editor to use)

`git commit`


One line commit

`git commit -m "My commit message"`

**Good Commit Messages**

[comment]: <> (TODO: Add what is a good commit message content here)

# Updating (amending) a Commit #

If you mess up a commit and want to change or add a file use `git commit --amend` as in the example below.  Your configured editor will pop up to allow you to modify the commit message as well.

    git commit -m 'My commit message'
    git add forgotten_or_updated_file
    git commit --amend

***WARNING:*** *Only amend commits that are in your local and have not been pushed to a remote repository.  Amending previously pushed commits and force pushing the branch will cause problems for your collaborators.*

[comment]: <> (TODO: Need a link here to the remote workflow sections.)

# Unstaging a Staged File #

If you staged a file by mistake you can un stage it with the command below.  Git status shows this command as a hint as well.

`git reset HEAD filename.xyz`

# Unmodifying a Modified File #

If you want to undo the changes you made to a tracked file use the command below.  

***WARNING*** *This is a dangerous command.  Git will replace the specified files witht he last **staged or committed** version.  Any changes you made to the file will be gone*

`git checkout filename.xyz`