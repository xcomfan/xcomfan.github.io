---
layout: page
title: "Repository history"
permalink: /gitref/history/
---

# Viewing the Commit History #

The **git log** command command will show you the history below are some useful modifiers to the git log command.

* `git log [--name-only | --name-status  ]` - Shows just the list of files modified after commit information with name-status adding the added, modified, deleted information.

* `git log -p` or `git log --patch` will show the patch output introduced in each commit.

* `git log --pretty` changes the log output to formats other than the default.  There are some pre pubit options available.

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

* `git log --relative-date` Show the "2 weeks ago" format insted of the full date.

[comment]: <> (TODO: What is a tree hash?)
[comment]: <> (TODO: Make a pass over this section and see if there is a better way to organize it.)
[comment]: <> (TODO: Some of the tables on this page are not adjusting to page width see if you can fix that in Jekyll options)
[comment]: <> (TODO: Clarify how to distinguish between a merge commit and a regular commit)

## Git log and branching ##

By default git log will only show history for the branch you are currently working on.  To see the history for another branch you can explicitly specify it.

`git log <some_branch_name>`

You can also have Git show you the history for all branches

`git log --all`

# Limiting and Filtering git log Output

| Option | Description | Example |
| ------ | ----------- | ------- |
| -n | Showns only the last n commits | `git log -2`|
| --since, --after, --until, --before | Limit commits to those after a specific date | `git log --since=2.weeks`, `git log --since="2022-01-01"`,  `git log --since="2 years 1 day 3 minutes ago"`|
| --author, --committer | Only shows commits where in which author or committer matches | `git log --author="John Doe"` |
| --grep | Only show commits with a commit message containing the string.  You can have multipe --grep arguments and you can use the --all-match or --invert-grep option to control if you match on all or one. | `git log --grep="testing"` ,  `git log --grep="Some String" --regexp-ignore-case` |
| -S | Only show commits adding or removing code matching the string.  Referred to as the "Pickaxe" feature | `git log -S my_function` |
| --no-merges | Prevent the display of merge commits | `git log --no-merge` |
| -- path/to/file | Limit output to commit that itroduced changes to those files | `git log -- a.c`, `git log -- *.c`|