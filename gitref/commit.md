---
layout: page
title: "Committing Your Changes"
permalink: /gitref/commit/
---

## Basic Commit

Use default editor to write your commit message.

[comment]: <> (TODO: Add link to config section where you specify what editor to use)

`git commit`


One line commit

`git commit -m "My commit message"`

**Good Commit Messages**

[comment]: <> (TODO: Add what is a good commit message content here)

## Updating (amending) a Commit

If you mess up a commit and want to change or add a file use `git commit --amend` as in the example below.  Your configured editor will pop up to allow you to modify the commit message as well.

    git commit -m 'My commit message'
    git add forgotten_or_updated_file
    git commit --amend

***WARNING:*** *Only amend commits that are in your local and have not been pushed to a remote repository.  Amending previously pushed commits and force pushing the branch will cause problems for your collaborators.*

[comment]: <> (TODO: Need a link here to the remote workflow sections.)

## Unstaging a Staged File

If you staged a file by mistake you can un stage it with the command below.  Git status shows this command as a hint as well.

`git reset HEAD filename.xyz`

If on git version 2.23.0 or newer you can also use the git restore command

`git restore --staged filename.xyz`

## Un-modifying a Modified File

If you want to undo the changes you made to a tracked file use the command below.  

***WARNING*** *This is a dangerous command.  Git will replace the specified files width he last **staged or committed** version.  Any changes you made to the file will be gone*  If you wish to keep the changes made to the file but get it out of the way temporarily you can stash it.

`git checkout filename.xyz`

If on git version 2.23.0 or newer you can also use the git restore command

`git restore filename.xyz`

## Interactive staging

Interactive staging is useful if you want to stage parts of files to make a more logical commit.

To use interactive staging use the -i or --interactive arguments to git add.  

`git add -i`

Once you run this command you will get an interactive prompt which will allow you to choose what you want to add to the commit.  Use the update option if you want to stage an entire file.  Use the patch option if you want to pick portions of a file to stage.

When you pick the patch option you will be presented with hunks of a file and given the ability to stage the hunk not stage it or to split it into smaller hunks or to manually edit the hunk.

When using the edit option to edit a hunk of file to not include a removed line change the - to a space.  To remove an added line from the commit just delete it in the interactive editor.

After you exit the interactive staging you can run `git commit` to make your commit.

## Cleaning your working directory

The git clean command is useful for removing untracked files from your working directory.  ***Note:*** files removed with this command cannot be retrieved.

To remove any untracked files and also any subdirectories that become empty use the command

`git clean -f -d`

The -f option is the force option and -d will make it recourse.

By default git clean will not operate on files that are in teh .gitingore file.  You can use the -x option to have it remove those files as well.  This is useful for removing build artifacts in a script.