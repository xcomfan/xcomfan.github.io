---
layout: page
title: "Tagging"
permalink: /gitref/tagging/
---

[comment]: <> (TODO: REV MARKER)

## What are tags

Tags are typically used to mark release points.  They can also be used to mark certain points in a repository history as important.

Git supports two type of tags.

* A lightweight tag is very much like a branch that doesn't change.  Its just a pointer to a specific commit.

* Annotated tags are stored as full objects in the Git database.  They are checksummed; contain the tagger name, email and date; have a tagging message; and can be signed and verified with GNU Privacy Guard (PGP).  Using annotated tags is recommended.

Tags are not transferred to remote servers with the git push command by default.  You need to explicitly push tags to a shared server.

## Working with tags

[comment]: <> (TODO: For some reason the level 2 tags are bigger than level 1 need to look into that.)

### Listing your tags

To list all the tags in a repo

`git tag`

To search for tags that match a particular pattern use the -l option

`git tag -l "v1.8.4*`

### Creating tags

To create an annotated tag use the -a option.  The -m specifies a tagging message which is stored with the tag.  If you don't specify a tag message Git will launch an editor for you.

`git tag -a v1.4 -m "My version 1.4`

To tag a specific commit.

`git tag -a v1.2 9fceb02`

To create a lightweight tag don't apply any of the -a -s or -m options and just provide a tag name

`git tag v1.4.lightweighttag`

### Viewing annotated tag details

To view tag data along with the commit that was tagged use gi show.  If you use git show on a lightweight tag, you will just see that the commit was tagged.

`git show v1.4`

### Checking out tags

To view the version of your code a tag is pointing to use the command.

`git checkout <tagname>`

[comment]: <> (TODO: This is a good info and it may be a good idea to have that someplace else and link to it instead of just having it in this file.  Also this is a good use case to work though to get a better understanding.)

After running this command you will see a message stating you are in a detached HEAD state.  If you make changes or create a commit in this state, the tag will stay the same, but your new commit won't belong to any branch and weill be unreachable, except by the exact commit hash.  Thus if you need to make changes such as if you are fixing an older version you will generally need to create a branch.

### Sharing tags

To push a tag to a remote (they are not pushed via git push by default)

`git push origin <tagname>`

To push multiple tags use the --tags option.  This will push all your tags to a remote sever.  This command will push both lightweight and annotated tags.

`git push origin --tags`

### Deleting tags

To delete a tag in your local repository use the command below.  

`git tag -d <tagname>` ***Note:*** this will not delete the tag from the remote server.

To delete a tag from a remote server

`git push origin --delete <tagname>`