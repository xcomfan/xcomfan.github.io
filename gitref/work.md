---
layout: page
title: "Working on your code"
permalink: /gitref/work_locally/
---


[comment]: <> (TODO: Sections of This page would work better as a mermaid diagram with you being able to click in to the stage un stage commit)

[comment]: <> (TODO: Need to break up this section into smaller subsection.)

## Basic Concepts

### The three trees of Git

 A Git project has three trees (tree here means a collection of files not the data structure) that it manages.  

#### The working directory (working tree)

The working directory (also called the working tree) is a single checkout of one version of the project.  You can think of the working directory as a sandbox where you can try changes before promoting them to the staging area (index) and eventually committing them.  Internally Git stores files in a compressed format, but for the working directory it extracts them onto the disk for you and makes them available in the working so that they can be easily modified.

#### The staging area (index)

The index/staging area is where you place your changes in order to commit them.  When you use the Git commit command, Git converts the changes that you staged (place in staging area) into a commit.

[comment]: <> (TODO: Good idea to take some of the content from the internals section and cover exactly what a commit is here.)

#### The HEAD

The HEAD "tree" is a pointer to the current branch reference, which in turn is a pointer to the last commit on that branch.  You can think of HEAD as the snapshot of your last commit on the branch you are currently working on.

### The 3 states of your files in Git

As you are working locally in your Git project, the files in your working directory can be in one of the 3 states below.  These 3 states correspond to one of the three trees of Git above.

| State | Description | Corresponding Tree |
| ----- | ----------- | ------------------ |
| modified | You have changed the file, but have not yet committed it to the database | working tree |
| staged | You have marked a file in its current version to go into your next commit snapshot | index |
| committed | The file state is safely stored in your Git database | HEAD |

## Git local workflow

{% mermaid %}

flowchart TD

modify([Modify files])
undo_mod([restore modified file to last committed state])
stage([Stage changes for next commit])
unstage([Un-stage a file])
commit([Commit files in the staging area])
undo_committ([Undo commit and keep changes])
reset_hard([Undo commit and discard changes])

modify-->stage
modify-.optional.->undo_mod
undo_mod-->modify
stage-->commit
stage-.optional.->unstage
unstage-->modify
commit-.optional.->undo_committ
undo_committ-->modify
commit-.optional.->reset_hard
reset_hard-->modify

{% endmermaid %}

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

## Working with git reset

### Git concepts to help understand git reset command



##### Git workflow of the "three trees"

Git's typical workflow is to record snapshots of your project in successively better states, by manipulating the states of the three trees.

Lets say we start with a new directory which is not yet a Git repository with a single file in it.  If we run the `git init` command this will create a Git repository with a **HEAD** reference pointing to the unborn *master* branch.  At this point only the **working directory** has any content.

Now we want to commit our single file, so we use the `git add` command.  This will take the content in our **working directory** and copy it to the **index**.

We then run `git commit`, which takes the contents of the **index** and saves it as a permanent snapshot, creates a commit object which points to that snapshot, and updates *master* to point to that commit.  At this point if you run `git status` you will see no change because all three trees are the same.

Switching  branches or cloning goes though a similar process. When you checkout a branch, it changes **HEAD** to point to the new branch ref, populates your **index** with the snapshot of that commit, then copies the contents of the **index** into your **working directory**.

### The role of the reset

When you run the `git reset` command, Git manipulates the three trees doing  up to three operations depending on the options specified.

#### Step 1: Move the HEAD (--soft)

The first thing reset will do is move what **HEAD** points to.  This is not the same as changing HEAD itself (which is what checkout does); reset moves the branch that **HEAD** is pointing to.

All calls to git reset have this as the first step.  If you specify the --soft argument, git reset will stop here.  This essentially undoes the last commit.    At this stage if you run `git commit`, Git creates a new commit and moves the branch that HEAD points to up to it.  When you reset back to HEAD~ (the parent of HEAD), you are moving the branch back to where it was, without changing the index or working directory (The working directory and index stay the same, but you won't see the previous commit in `git log`).

At this point you can run `git commit` again to accomplish what `git commit --amend` would have done.

#### Step 2: Updating the index (--mixed)

Step two is the default for git reset.  If you run git reset without any argument it will stop at this step 2.  In step 2 reset will update the index with the contents of whatever snapshot HEAD points to.  The --mixed argument is optional.

At this step your last commit is still undone, but it will also un-stage everything. In other words `git reset HEAD~` will undo your last commit and add commands.

#### Step 3: Updating the working directory (--hard)

The third ting that reset will do (if you specify the --hard) option is to make the working directory look like the index.

In other words git `reset --hard HEAD~` will undo the git commit command your for your prior commit, the git add command for your prior commit, and eliminate any changes you made to the files in your working directory.

### Git reset with a path

You can also provide git reset with a path to act on.

If you specify a path, reset will skip step 1 and limit the remainder of its actions to a specific file or set of files.  This makes sense because HEAD is a pointer and you can't point to a part of a commit, but index and working directory can be partially updated.

[comment]: <> (TODO: book mentions that you can run reset with a SHA-1 or branch. Need to try the branch option)

If you run the command `git reset file.txt` (this command is essentially a short hand for `git reset --mixed file.txt`) git reset will

1. skip moving the branch HEAD points to
2. make the index look like HEAD

In other words it copies file.txt from HEAD to index.  This has the practical effect of unstaging the file.

You don't have to pull the file from HEAD you can specify the commit to pull that version from.  `git reset eb43b4 file.txt`

You can use the --patch option with git reset to un-stage content on a hunk by hunk basis.

### Using git reset for squashing

[comment]: <> (TODO: Link this section to the interactive rebase squashing section and we probably need a mermaid section fro squashing in some workflow.)

Lets say you have a project where...

* The first commit has one file.
* The second commit added a new file and changed the first.
* The third commit changed the first file again.

If the second commit was a work in progress and you want to squash it down you can...

run the command `git reset --soft HEAD~2` to move the HEAD branch back to an older commit (the most recent commit you want to keep)

then simply runt `git commit` again.

This would remove the second commit from your reachable history (the history you would push)

## Reset versus checkout

### Without path specified

Running `git checkout [branch]` is pretty similar to running `git reset --hard [branch]` in that it updates all 3 trees for you to look like `[branch]`.

There are two important differences however.

1. Unlike `reset --hard`, `checkout` is working directory safe.  It will check to make sure its not blowing away files that have changes to them.  It actually tries to do a trivial merge in the working directory os all the files you have not changed will be updated.  `reset hard` on the other hand, will simply replace everything across the board without checking.

2. The way HEAD is updated is different.  `reset` will move the branch that HEAD points to, `checkout` will move HEAD itself to point to another branch.  So for instance if we have master and develop branches which point to different commits, and we are currently on the develop branch (so HEAD points to it).  If we run `git reset master`, develop itself will now point to the same commit that master does.  If we instead run `git checkout master` develop does not move, HEAD itself does.  HEAD will now point to master.  In both cases we are moving HEAD to point to commit A, but how we do so is very different.  `reset` moves the branch HEAD points to, `checkout` moves HEAD itself.

### With path specified

If you run `checkout` with a file path.  Like `reset` this does not move HEAD.  `checkout` will update the index with that file at that commit, but it will also overwrite the file in the working directory.

[comment]: <> (TODO: This is not much of a comparison, just gives you a what checkout does tidbit.  Should move somewhere else when I reorder thins!)

You can use the --patch option with checkout to select hunks you wish to check out.