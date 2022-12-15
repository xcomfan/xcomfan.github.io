---
layout: page
title: "Repository history"
permalink: /gitref/history/
---

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