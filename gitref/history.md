---
layout: page
title: "Repository history"
permalink: /gitref/history/
---

[comment]: <> (TODO: REV MARKER)

[comment]: <> (TODO: This file is a bit big.  Need to split it up into more manageable chunks?)

## Viewing the Commit History

The **git log** command command will show you the history below are some useful modifiers to the git log command.

* `git log [--name-only | --name-status  ]` - Shows just the list of files modified after commit information with name-status adding the added, modified, deleted information.

* `git log -p` or `git log --patch` will show the patch output introduced in each commit.

* `git log --pretty` changes the log output to formats other than the default.  There are some predefined options available.

    * `git log --pretty=oneline` will print each commit on a single line.

        * `git log --pretty=oneline --graph` Adding the graph option will add an ASCII graph showing the branch and merge history which combines well with the --pretty=oneline modifier.

    * `git log --pretty=format:"%h - %an, %ar : %s"` lets you customize the output format where...

        * | Specifier | Description of Output|
          | --------- | -------------------- |
          | %H , %h| Commit hash, abbreviated commit hash |
          | %T , %t | Tree hash, abbreviate tree has | 
          | %P , %p | Parent hashes, abbreviated parent hashes |
          | %an , %ae | Author name, Author email |
          | %ad , %ar | Author date (format respects the --date=option), Author date, relative |
          | %cn , %ce | Committer name, Committer email |
          | %cd , %cr | Committer date, Committer date relative |
          | %s | Subject |

    *  useful example of custom output: `git log --pretty=format:"%h %cd %cn %s" --graph`

    * `git log --pretty=[short |full | fuller]` lets you see various amounts of information. (Author, Date, Commit Date, Author Date etc.)

* `git log --stat` will show the commit stats (number of changes etc)
    * `git log --shortstat` Displays only the changed/insertion/deletions line from the --stat command.

* `git log --relative-date` Show the "2 weeks ago" format instead of the full date.

[comment]: <> (TODO: What is a tree hash?)
[comment]: <> (TODO: Make a pass over this section and see if there is a better way to organize it.)
[comment]: <> (TODO: Some of the tables on this page are not adjusting to page width see if you can fix that in Jekyll options)
[comment]: <> (TODO: Clarify how to distinguish between a merge commit and a regular commit)

## Git log and branching

By default git log will only show history for the branch you are currently working on.  To see the history for another branch you can explicitly specify it.

`git log <some_branch_name>`

You can also have Git show you the history for all branches

`git log --all`

## Limiting and Filtering git log Output

| Option | Description | Example |
| ------ | ----------- | ------- |
| -n | Shows only the last n commits | `git log -2`|
| --since, --after, --until, --before | Limit commits to those after a specific date | `git log --since=2.weeks`, `git log --since="2022-01-01"`,  `git log --since="2 years 1 day 3 minutes ago"`|
| --author, --committer | Only shows commits where in which author or committer matches | `git log --author="John Doe"` |
| --grep | Only show commits with a commit message containing the string.  You can have multiple --grep arguments and you can use the --all-match or --invert-grep option to control if you match on all or one. | `git log --grep="testing"` ,  `git log --grep="Some String" --regexp-ignore-case` |
| -S | Only show commits adding or removing code matching the string.  Referred to as the "Pickaxe" feature | `git log -S my_function` |
| --no-merges | Prevent the display of merge commits | `git log --no-merge` |
| -- path/to/file | Limit output to commit that introduced changes to those files | `git log -- a.c`, `git log -- *.c`|

## Git log searching

If you are not looking for where a term exists, but when it existed or was introduce you want to use git log searching.

If for example you want to find when the ZLIB_BUF_MAX constant was originally introduced, we can use the -S option to git log.  The output of the command will be the commit details of the commit that introduced the variable. If you need to be more specific you can use the -G option to provide a regular expression.

`git log -S ZLIB_BUG_MAX --oneline`

### Line Log Search

you can use Line Log Search to check the history of a function or a line of code.  Just use the -L option with git log.  For example, if we want to see every change made to the function git_deflate_bound in the zlib.c file we would do similar to the following command.  This will try to figure out what the bounds of that function are and then look though the history and show us every change that was made to the function as a series of patches.

`git log -L :git_deflate_bound:zlib.c`

[comment]: <> (TODO: Play around with this feature to get a better write up and understand it better?)

If Git can't figure out hwo to match a function or method in your programming language, you can also provide it with a regular expression as in the example below.

`git log -L '/unsigned long git_deflate_bound/',/^}/:zlib.c`

## Revision Selection

Git allows you to refer to a single commit, set of commits, or range of commits in a number of ways.

### Single Revisions

You can refer to a commit using the full 40 character SHA-1 of the commit, but typically the first few characters of the sha are needed.  4 is the minimum, but 8 to 10 characters will be enough to make sure your selection is unique.

### Branch References

If a commit is the tip of a branch, you can use teh branch name in any git command that expects a reference

`git show ca82a6dff812ec44f44442008202890a8989033`

`git show topic1`

To get the commit hash of a branch you can use the rev-parse command.

`git rev-parse topic1`

### Reflog Shortnames

As you are working, Git keeps a "reflog" a log of where your HEAD and branch references have been.  Every time your branch tip is updated for any reason, Git will store that information for you in this temporary history.  ***Note:*** reflog information is strictly local.  Reflog will only contain information for what you have done in your local repository.  Think of reflos as Git's version of the Linux shell history.

To see this history use the command 

`git reflog`

You can use reflog data to refer to older commits.

If you want to see the 5th prior commit

`git show HEAD@{5}`

You can also look back days, months etc

`git show HEAD@{2.months.ago}`

To see reflog information formatted like the git log output use the -g option to git log

`git log -g master`

### Ancestry References

[comment]: <> (TODO: This explanation sucks play around with these commands and write a better one.?)

You can specify commits via their ancestry.  If you place a ^ (caret) symbol at the end fo a reference, Git resolves it to mean the parent of that commit.  You can also specify a number after the ^ at the end of a reference to identify which parent you want.  For example d921970^2 means the second parent of commit d921970.

The other ancestry specification is the ~ (tilde) symbol.  This reference also refers to the parent, so HEAD~ and HEAD^ are equivalent.  HEAD~2 means the first parent of the fist parent.

You can combine the two references to do something like HEAD~3^2

### Commit Ranges

If you have a lot of branches, you can use Git's range specification to answer questions such as "What work is on this branch that I haven't yet merged into my main branch?"

#### Double Dot range specification

Given the commit history below 

{% mermaid %}
 flowchart RL

    F([F])-->E([E])-->B([B])-->A([A])
    D([D])-->C([C])
    C-->B
    experiment[experiment]-.-D
    master[master]-.-F

{% endmermaid %}

Say you want to see what is in your experiment branch that has not yet been merged into your master branch.  You can ask git to show you a lof of just those commits with the master..experiment range.

`git log master..experiment`

This command means all commits reachable from experiment that aren't reachable from master. So for the above example you would have D and C as the result.

[comment]: <> (TODO: Try this in an actual repo and update notes with your findings)

If on the other hand you wanted to see the opposite: all commits in master that are not in experiment you reverse the double dot range.

`git log experiment..master`

This would yield the F and E commits.

A useful take on this command is if you want to check what you are about to push.  The command below will give you a summary of what you are about to push if your branch is tracking origin/master

`git log origin/master..HEAD`

#### Multiple points range specification

If you want to see what commits are in any of several branches that aren't in the branch you are currently on, Git allows you to do this by suing either the ^ or the --not before any reference from which you don't want to see reachable commits.  The following commands are equivalent.

[comment]: <> (TODO: Another crappy explanation that needs some experimenting and updating)

`git log refA..refB`

`git log ^refA refB`

`git log refB --not refA`

This syntax can specify more than two references in your query which you cannot do with double dot syntax.  For example if you want to see commits that are reachable from refA or refB but not from refC you can use either of the commands below.

`git log refA refB ^refC`

`git log refA refB --not refC`

#### Triple Dot range specification

The triple dot range selection syntax specifies all the commits that are reachable by either of two references but not by both of them.

If you want to see what is in master or experiment but not any common references you can run the command

`git log master...experiment`

This command will give you the normal git log output.  You can modify that with --left-right option which will show which side of the range each commit is in and make the output easier to read.

`git log --left-right master...experiment`

In your sample history this would yield

< F

< E

\> D

\> C

[comment]: <> (TODO: Need to play around with this feature as well)

## Rewriting commit history

### Interactive rebase

[comment]: <> (TODO: Write up how to use interactive rebase here so the section below can just cover the specific actions?)

If your want practice with interactive rebase go to [git-rebase.io](https://git-rebase.io)

### Changing the last commit

If you just want to change your commit message (in your local working directory) the command below will bring up the editor for you to do so if you have no changes staged.

`git commit --amend`

If you have changes staged, the command above will modify your last commit again in your local working directory with the staged changes and allow you to modify your commit message.

If you want to modify the code of your last commit with the staged changes, but not the commit message use the command

`git commit --amend --no-edit`

***Note:*** Amending a commit changes the SHA-1 of the commit.  **DO NOT AMEND YOUR LAST COMMIT IF YOU ALREADY PUSHED IT.**

### Squashing Commits

You can use the interactive rebase to take a series of commits and squash them into a single commit.

First run the rebase command to go back the desired amount of commits

`git rebase -i HEAD~3`

In the interactive rebase editor replace the `pick` keyword with `squash` and Git will apply both that change and the change directly before it and have you merge the commit message.

[comment]: <> (TODO: Try this out and make any clarification updates also need to take the changing multiple commit below details of the step by step process and apply it here since you are placing this one first.)

### Splitting Commits

Splitting a commit undoes a commit and then partially stages and commits as many times as commits you want to end up with.  For example if you want to split the middle commit of the three commits in the example below you can do that with the rebase -i script.

In the script generated by the `rebase -i` command change the instruction on the commit you wish to split to `edit`.

When the script drops you to the command line, you reset that commit, take the changers that have been reset, and create multiple commits out of them.  

When you save and exit the editor, Git will rewind to the parent of the fist commit in your list, apply the first commit, apply the second and drop you to the console.  Then you can do...

`git reset HEAD^` - resets the second commit (the one you want to split)

`git add README` - first of your split up changes

`git commit -m 'Update README formatting` - first of your split commits

`git add lib/simplegit.rb` - second of your split up changes

`git commit -m 'Add blame'` - second of your split up commits

`git rebase --continue` - apply the third commit from the rebase script

***Note:*** Do not do this with commits you already pushed

### Deleting a Commit

Interactive rebase can also delete a commit.  

In the script that interactive rebase has you edit, put the word drop before the commit you want to delete.

Because of the way Git builds commit objects, deleting a commit will cause the rewriting of all the commits that follow it.  The further back you go, the more commits need to be created which can cause lots of merge conflicts.  If you get partway through a change like this and want to abort you can use the command below to return your repo to its state before you attempted the rebase.

`git rebase --abort`

If you finish a rebase and decide its not what you want, you can use git reflog to recover an earlier version of your branch.

[comment]: <> (TODO: Need a link to reflog details also put this at top in section that covers how to use rebase as it applies to all these changes.?)

### Changing multiple commit messages

Git does not have a modify history tool, but you can use the rebase tool to rebase a series of commits onto the HEAD that they were originally based on.

With the interactive rebase tool you can stop after each commit you want to modify and change the message, add files or do whatever you wish.

For example if you want to change the last 3 commits you would use the command

`git rebase -i HEAD~3`

The -i option is for interactive rebase

***Note:*** This is a rebase command so **DO NOT** modify commits you already pushed.

Once you run the above command will bring up an editor with a list of commits you selected listed newest first. The editor shows a script the rebase will run as well as some hint information.

You need to edit this script replacing the word `pick` with `edit` on the commits you wish to stop at.

When you save and exit the editor, Git will rewind you to the last commit in the list and drops you at the command line with some hint information displayed.

As the hint says, you can now use `git commit --amend` to modify the commit you landed on or `git rebase --continue` to move on to the next commit you marked with the `edit` word in the script.

Once you are done modifying all the selected commits you will have a rewritten history.

### Reordering Commits

Similar to rewriting multiple commits, you can use interactive rebase to reorder or remove commits entirely.  In the example below if you want to remove the "Add cat file" commit and reorder the other two you would take the below interactive rebase script

```
pick f7f3f6d Change my name a bit
pick 310154e Update README formatting and add blame
pick a5f4a0d Add cat-file
```

and change it to

```
pick 310154e Update README formatting and add blame
pick f7f3f6d Change my name a bit
```

When you save and exit the editor, Git rewinds your branch to the parent of these commits, applies 310154e and then f7f3f6d and then stops.

At this point you effectively changed the order of those commits and remove the "Add-cat-file" commit completely.

***Note:*** This is a rebase command so **DO NOT** modify commits you already pushed.

## The Nuclear Option: filter-branch

The filter-branch option will rewrite huge swathes of your commit history.  **Do not use it unless your project is not public or is worked on by a few people**

There is an alternative to filter-branch which you can find [here](https://github.com/newren/git-filter-repo)

[comment]: <> (TODO: Need to experiment with script above and write that up)

If you are going to use filter-branch it is recommended you try is a backup copy of your repository.

### Removing a file from every commit

This is useful if someone commits a giant binary file by mistake, or if someone commits a file with a password and you want to make the project public.

To remove a file called password.txt you would use the command below.  This command will remove the file passwords.txt from every commit whether it exists or not. 

`git filter-branch --tree-filter 'rm -f passwords.txt' HEAD`

The `--tree-filter` option runs the specified command after each checkout of the project and then recommits the result.

If you want to run filter-branch on all of your branches you can pass the `--all` command.

### Changing email address globally

The command below can globally update your email address.  You need to be careful to change only the email addresses that are yours so use the `--commit-filter`.  **This command will change every single commit in your history not just those that hve the matching email address.**

### Making a subdirectory the new root

Suppose you have imported from another source control system and have subdirectories that make no sense (trunk, tags, etc.).  If you want to make the trunk subdirectory the new project root for every commit you can use the below command.

`git filter-branch --subdirectory-filter trunk HEAD`

```
git filter-branch --commit-filter '
        if [ "$GIT_AUTHOR_EMAIL" = "myemail@mydomain" ];
        then
            GIT_AUTHOR_NAME="Your Name";
            GIT_AUTHOR_EMAIL="myemail@mydomain";
            git commit-tree "$@";
        else
            git commit-tree "$@";
        fi' HEAD
```

Now your new project root is what was in the trunk subdirectory.  Git will also automatically remove commits that did not effect the subdirectory.

[comment]: <> (TODO: The below is good intro content that needs to not be buried in this file)

## Working with git reset

### Git concepts to hep understand git reset command

#### The three trees

An easier way to think about rest and checkout is though the mental model of Git managing three different trees.  Trees here refer to a collection of files not the data structure.

##### The HEAD

* The HEAD "tree" is the last commit snapshot and the next parent.
* HEAD is the pointer to the current branch reference, which is in turn a pointer to the last commit made on that branch.
* You can think of HEAD as the snapshot of your last commit on your current branch

##### The Index

* The index is your proposed next commit.  Its also referred to as the "staging area"
* The index is what Git looks at when your run the git commit command.
* Git populates the index with a list of all the file contents that were last checked out into your working directory and what they looked like when they were originally checked out.
    * You then replace some of these files with new versions and the git commit command converts them into a new commit.

##### The working directory

* Think of the working directory as a sandbox where you can try changes before committing them to the index/staging area.
* The other two trees store their content in an efficient, but inconvenient manner.  The working tree unpacks them into actual files making them easier to edit.
* Working directory is commonly called the "working tree"

##### Git workflow of the "three trees"

Gits typical workflow is to record snapshots of your project in successively better states, by manipulating the states of the three trees.

Lets say we start with a new directory which is not yet a Git repository with a single file in it.  If we run the `git init` command this will create a Git repository with a **HEAD** references pointing to the unborn *master* branch.  At this point only the **working directory** has any content.

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

## Debugging with Git

### File annotation (git blame)

File annotations is the best tool if you want to track down when something such as a bug was introduced to the code and why.  File annotations will show you what commit was the last to modify each line of any file.  So for example if you see a method in your code is buggy you can annotate it with git blame to determine which commit was responsible for the introduction fo that line.

`git blame -L 69,82 problemfile.txt`

The -L option allows you to specify a range of lines to display the git blame output for.

The resulting output will show.

* partial SHA-1 of the commit in first column.
* committer and commit date followed by line number in second column
* a carat (^) symbol at the beginning of the commit SHA-1 indicates that the content was there from the initial commit.

[comment]: <> (TODO: Experiment with teh bit below to better understand it)

You can use git blame with the -C option to try and figure our where snippets of the code withing it originally came from.  This is useful if there was a code refactor.  Git will not just find where the line are but the history of the original location they came from.

`git blame -C -L 123, 145 somefile.txt`

### Binary Search (git bisect)

The bisect command does a binary search though your commit history to help you identify as quickly as possible which commit introduced an issue.

The workflow is that you use git bisect to start things off then mark the current version as bad.  Then you tell Git the last good commit.  Git will then binary search though the commits in between to help you identify which commit  was the bad one.  

```
git bisect start
git bisect bad
git bisect good v1.0
```

You can keep using `git bisect good` and `git bisect bad` to test the various commits Git rolls you back to in the binary search.

When you are done with your search, you can use `git bisect reset` to reset your HEAD to where it was before you started.

If you have a script that will exist with a 0 if your project is good or non zero if bad you can automate the process.

```
git bisect start HEAD v1.0
git bisect run test-error.sh
```

Doing this automatically runs test-error.sh on each checked out commit until Git finds the first broken commit.