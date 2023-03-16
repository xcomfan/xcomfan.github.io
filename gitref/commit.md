---
layout: page
title: "Committing Your Changes"
permalink: /gitref/commit/
---

[comment]: <> (TODO: REV MARKER)

## Basic Commit

Use default editor to write your commit message (Git will bring up the editor when you run the command).

[comment]: <> (TODO: Add link to config section where you specify what editor to use)

[comment]: <> (TODO: Add how to undo a commit completely if you made one by mistake.)

`git commit`

Commit and comment in single command.

`git commit -m "My commit message"`

**Good Commit Messages**

[comment]: <> (TODO: Add what is a good commit message content here)

## Updating (amending) a Commit

If you mess up a commit and want to change or add a file use `git commit --amend` as in the example below.  Your configured editor will pop up to allow you to modify the commit message.

```
git commit -m 'My commit message'
git add forgotten_or_updated_file
git commit --amend
```

***WARNING:*** *Only amend commits that are in your local repository and have not been pushed to a remote repository.  Amending previously pushed commits and force pushing the branch will cause problems for your collaborators.*

[comment]: <> (TODO: Need a link here to the remote workflow sections.)