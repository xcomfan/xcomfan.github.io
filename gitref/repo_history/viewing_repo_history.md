---
layout: page
title: "Viewing Repository History"
permalink: /gitref/repo_history/viewing_repo_history
---

[comment]: <> (TODO: This file is a bit big.  Need to split it up into more manageable chunks?)

## Viewing the Commit History

The `git log` command command will show you the history below are some useful modifiers to the git log command.

## Git log and branching

By default git log will only show history for the branch you are currently working on.  To see the history for another branch you can explicitly specify it.

`git log <some_branch_name>`

You can also have Git show you the history for all branches

`git log --all`

## See which files were changed in each commit

`git log [--name-only | --name-status  ]` - Shows the list of files modified after displaying basic commit information.  name-status adds information for each file of whether the file was added, modified, or deleted in the commit.

[comment]: <> (TODO: This file is a bit big.  Need to link to chunk explanation here?)

## See the changes made (the patch) in each commit

`git log -p` or `git log --patch` will show the patch output introduced in each commit.

if you want to see the changes for a specific file use

`git log -p -- filename`

If you want to see relative dates ("2 weeks ago" for example) instead of full dates use the --relative-date flag.

`git log --relative-date`

## Alternate ways to display git log infromation (--pretty option)

`git log --pretty` changes the log output to formats other than the default.  There are some predefined options available.

`git log --pretty=oneline` will print each commit on a single line.

`git log --pretty=oneline --graph` Adding the graph option will add an ASCII graph showing the branch and merge history which combines well with the --pretty=oneline modifier.

### Customize the format of --pretty modifier output

`git log --pretty=format:"%h - %an, %ar : %s"` lets you customize the output format where...

| Specifier | Description of Output|
| --------- | -------------------- |
| %H , %h| Commit hash, abbreviated commit hash |
| %T , %t | Tree hash, abbreviate tree has | 
| %P , %p | Parent hashes, abbreviated parent hashes |
| %an , %ae | Author name, Author email |
| %ad , %ar | Author date (format respects the --date=option), Author date, relative |
| %cn , %ce | Committer name, Committer email |
| %cd , %cr | Committer date, Committer date relative |
| %s | Subject |

`git log --pretty=format:"%h %cd %cn %s" --graph` - A useful example of using custom formats.  It may be a good idea to create an alias for this format.

`git log --pretty=[short |full | fuller]` lets you see various amounts of information. (Author, Date, Commit Date, Author Date etc.)

[comment]: <> (TODO: Some of the tables on this page are not adjusting to page width see if you can fix that in Jekyll options)
[comment]: <> (TODO: Clarify how to distinguish between a merge commit and a regular commit)

## Limiting and Filtering git log Output

| Option | Description | Example |
| ------ | ----------- | ------- |
| -n | Shows only the last n commits | `git log -2`|
| --since, --after, --until, --before | Limit commits to those after a specific date | `git log --since=2.weeks`, `git log --since="2022-01-01"`,  `git log --since="2 years 1 day 3 minutes ago"`|
| --author, --committer | Only shows commits in which author or committer matches | `git log --author="John Doe"` |
| --grep | Only show commits with a commit message containing the string.  You can have multiple --grep arguments and you can use the --all-match or --invert-grep option to control if you match on all or one. | `git log --grep="testing"` ,  `git log --grep="Some String" --regexp-ignore-case` |
| -S | Only show commits adding or removing code matching the string.  Referred to as the "Pickaxe" feature | `git log -S my_function` |
| --no-merges | Prevent the display of merge commits | `git log --no-merge` |
| -- path/to/file | Limit output to commit that introduced changes to those files | `git log -- a.c`, `git log -- *.c`|

## Show commit stats

`git log --stat` will show the commit stats (number of changes etc)

`git log --shortstat` Displays only the changed/insertion/deletions line from the --stat command.