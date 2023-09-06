---
layout: page
title: "Working In The Working Directory"
permalink: /gitref/work_locally/workdir
---

## Viewing the status of your working tree

`git status`

This command will display the following details

| Section | Purpose |
| ------- | ------- |
| "On branch" | Branch you are currently on TODO: Link to Branching section once you have it.  Probably need to rephrase "working on"|
| "No commits yet" | Only displayed if repository has no commits/history |
| "Changes to be committed" | The staged changes that will be committed when you run the `git commit` command |
| "Changes not staged for commit" | Modified files in your working copy that are not staged to be committed |
| "Untracked Files" | Files in your working copy that Git is not tracking |

## Recording Changes In Your Working Tree

### Stage new files or changes to tracked files

**Add (stage) a single file**

`git add somefile.txt`

**Add all modified and new files**

`git add .`

**You can also add with globing** 

`git add *.py`

***Note:*** Git stages files exactly as they are at the moment of staging.  This means that if you `git add` a file and then modify it you will see it in both "Changes to be committed" and "Changes not staged for commit" sections when you run the `git status` command.

### See what changed in staged files.

If you staged a file and want to see what was changed in the file instead of a regular `git diff` you need to use the `--cached` option of git diff.

`git dif --cached mystagedfile.txt`

### Interactive staging

Interactive staging is useful if you want to stage parts of files to make a more logical commit.

To use interactive staging use the -i or --interactive arguments to git add.  

`git add -i`

Once you run this command you will get an interactive prompt which will allow you to choose what you want to add to the commit.  

- Use the **update** option if you want to stage an entire file.
- Use the **patch** option if you want to pick portions of a file to stage.

[comment]: <> (TODO: Need to create definition for hunk)
When you pick the patch option you will be presented with hunks of a file and given the ability to stage the hunk not stage it or to split it into smaller hunks or to manually edit the hunk.

[comment]: <> (TODO: No edit option above need to run though this and update notes with my own experience and language)
When using the edit option to edit a hunk of file to not include a removed line change the - to a space.  To remove an added line from the commit just delete it in the interactive editor.

After you exit the interactive staging you can run `git commit` to make your commit.

### Un-stage a file

**If you staged a file by mistake you can un-stage it with the command**

`git restore --staged myfile.txt`

If on Git version older than 2.23.0 use the below command also works.

`git reset HEAD filename.xyz`

### Un-modifying a modified File

If you want to undo the changes you made to a tracked file use the command below.  

***WARNING*** *This is a dangerous command.  Git will replace the specified files width he last **staged or committed** version.  Any changes you made to the file will be gone*  If you wish to keep the changes made to the file but get it out of the way temporarily you can [stash it]({% link gitref/stashing.md %}).

`git checkout filename.xyz` or `git restore filename.xyz` if on Git version 2.23.0 or newer.