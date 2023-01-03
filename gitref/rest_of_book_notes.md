---
layout: page
title: "Raw Notes"
permalink: /gitref/rawnotes/
---
# Git Configuration

## Customizing Git - Git Configuration

### Git Configuration

This section was just a review of the 3 config levels I already have in notes.

#### Basic Client Configuration

Git configuration options fall into two catagories, client-side and server-side.  Majority of options are client-side.  There are many configuration options the common ones are covered here.   To see a list of all the options use the command below or [the reference here](link https://git-scm.com/docs/git-config)

`man git-config`

**core.editor**

Sets your editor to a value that is not derived from the VISUAL or EDITOR shell environment variables.

`git config --global core.editor emacs`

**commit.template**

If you set this to the path of a file on your system, Git will use that file as the default initial message when you commit.  You can use this to remind yourself or others the format and style when creating a commit message.

[comment]: <> (TODO: The book in this section has an example that tied into the good commit message discussion.  Make sure to link to this in that section of your notes.)

`git config --global commit.template ~/.gitmessage.txt`

**core.pager**

This setting controls which pager is used when git pages output such as log or diff.  You can set it to `more` or your favorite pager (by default it uses less) or you can turn it off by setting a blank value.  If off git will page the entire output regardless of how long it is.

`git config --global core.pager ''`

**user.signinkey**

This sets your GPG signing key for signed annotated tags.  You can set your key ID with the command

`git config --global user.signinkey <gpg-key-id>`

**core.excludesfile**

This setting is like the .gitignore file, but applies to all of your repositories.  This is a good place to put filenames such as .DS_Store (for Mac users) or .swp files.

`git config --global core.excludesfile ~/.gitignore_global`

**help.autocorrect**

By default if you miss type a command git will suggest the command for you.  If you set this option to 1 it will actually run the command it thinks you intended.

The value you actually set it to represents tenths of a second (10 means one second) and that is the amount of time Git will give you to change your mind if Git guesses the command wrong.

#### Colors in Git

**color.ui**

By default Git colors most of its output.  This lets you disable/enable that.

`git config --global color.ui false`

The default setting is auto which colors output if its going to terminal, but omits the color codes if its redirected to a pipe or file.  You can also set it to always to ignore the difference between terminal and pipes.  You can also pass --color to any Git command to force it to send the color codes ad hoc.

**color.\***

Git provides verb-specific color settings if you want to be more specific about which commands are colored.  Each of these can be set to `true`, `false` or `always`

* color.branch
* color.diff
* color.interactive
* color.status

In addition, each of these has a sub-setting you can use to set specific colors for parts of the output. For example to set the meta information in your diff output to blue foreground, black background and bold text use the command

`git config --global color.diff.meta "blue black bold"`

You can set the color to any of the following values.  `normal`, `black`, `red`, `green`, `yellow`, `blue`, `magenta`, `cyan` or `white`.  For emphasis attributes you have the options `bold`, `dim`, `ul` (underline), `blink` and `reverse` (swap foreground and background)

#### External Merge and Diff Tools

#### Formatting and Whitespace

#### Server Configuration

## Git Attributes

## Git Hooks

## An Example Git-Enforced Policy

## Summary

# Git and Other Systems

# Git Internals