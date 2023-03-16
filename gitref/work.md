---
layout: page
title: "Working on your code"
permalink: /gitref/work_locally/
---

[comment]: <> (TODO: REV MARKER)

`[comment]: <> (TODO: This page would work better as a mermaid diagram with you being able to click in to the stage un stage commit)`

## The 3 states of your files in Git

As you are working locally the files in your working directory can be in one of the 3 states below.

| State | Description |
| ----- | ----------- |
| modified | You have changed the file, but have not yet committed it to the database |
| staged | You have marked a file in its current version to go into your next commit snapshot |
| committed | The file state is safely stored in your Git database |

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

***WARNING*** *This is a dangerous command.  Git will replace the specified files width he last **staged or committed** version.  Any changes you made to the file will be gone*  If you wish to keep the changes made to the file but get it out of the way temporarily you can stash it.

`git checkout filename.xyz` or `git restore filename.xyz` if on Git version 2.23.0 or newer.


## Searching in your work

Git ships with a built in grep that allows you to search though any [committed tree]({% link gitref/definitions.md %}#committed-tree), or the index for a string of regular expressions.  The benefit of git grep over regular grep is that its faster and you can search through any [tree]({% link gitref/definitions.md %}#tree) in Git not just the working directory.

By default git grep will look through files in your local directory.  

**You can use the -n or --line-number option to print out the line numbers where Git has found matches.**

`git grep -n my_magic_function` or `git grep --line-number my_magic_function`

There are many options to git grep.  See `man git grep` for what's available.

**You can use -c or --count to get the file names and found instance counts.**

`git grep -c TODO` or `git grep --count TODO`

**Look for lines defining either of two options**

The example below will look for a line that has #define and either MAX_PATH or PATH_MAX.

`git grep -e '#define' --and \( -e MAX_PATH -e PATH_MAX \)`

**See the function name of the git grep match**

You can have git show you the preceding line of function matches with the -p or --show-function argument.  In other words Git Grep will look for function definition where your search matches and prints that so you know which function its in.   In example below I am searching for 'return' as its a good example of seeing the match and the function name.

`git grep -p return *.c` or `git grep --show-function return *.c`

You can search for complex combinations of strings with the --and flag, which ensures that multiple matches must occur in the same line of text. 

**Exclude a directory from your search**

The example below looks for solution, excluding files in Documentation.

`git grep solution -- :^Documentation`

**search for lines with values in a file that has all the listed values**

The example below looks for a line that has NODE or Unexpected in files that have lines that match both.
`git grep --all-match -e NODE -e Unexpected`

**search older version of code using a tag**

The example below will search for lines that define a constant whose name contains either of the substrings LINK or BUF_MAX specifically in an older version of the codebase by specifying the tag v1.8.0

`git grep --break --heading -n -e '#define' --and \( -e LINK -e BUF_MAX \) v1.8.0`

* --break argument will print empty lines between matches from different files

* --heading argument will show the file name above the matches

* -n or --line-number will add line numbers to output

* -e argument specified that the next parameter in command is the pattern.

## Cleaning your working directory

The git clean command is useful for removing untracked files from your working directory.  ***Note:*** files removed with this command cannot be retrieved.

To remove any untracked files and also any subdirectories that become empty use the command

`git clean -f -d`

The -f option is the force option and -d will make it recourse.

By default git clean will not operate on files that are in teh .gitingore file.  You can use the -x option to have it remove those files as well.  This is useful for removing build artifacts in a script.