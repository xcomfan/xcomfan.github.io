---
layout: page
title: "Git Configuration"
permalink: /gitref/gitconfigref
---

## 3 levels of configuration in Git

Git comes with a tool called git config that lets you get and set configuration variables that control various aspects of how Git looks and operates.  These configuration variables can be stored in 3 places.

* **System wide settings** - Controlled by `[patah]/etc/gitconfig` file - This file will contains values that are applied to every user on the system and all of their repositories.  As this is a system controlled file you need admin rights on the system to modify it.

* **Global (user level settings)** - Controlled by `~/.gitconfig` or `~/.config/git/config` - This file will have values specific to you the user.  You can have Git read and write to this file by using the `--global` option and it will effect all of the repositories that you work with on the system.

* **Local (repository level settings)** - Controlled by config file in the Git directory (that is `.git/config`) - These variables will apply  to a single repository.  You can have Git read and write to this file by using the `--local` option.  This is also the default file where settings are made.

Each level overrides values in the previous level, so values in `.git/config` will trump those in `[path]/etc/gitconfig`

## Viewing your Git settings

To view your setting and where they are coming from use the command

`git config --list --show-origin`

To view the value of a specific setting

`git config <setting>` for example `git config user.name`

## Modifying your Git settings

Set your name and email globally

`git config --global user.name "John Doe"`

`git config --global user.email johndoe@example.com`

Set your preferred editor

`git config --global core.editor vim`

Set the default name for your main/master branches (supported with Git version 2.28 and newer)

`git config --global init.defaultBranch main`

## The .gitignore file

You can create a .gitignore file in your project directory to tell Git to ignore certain files so that you don't accidentally commit them.

The rules for the patterns you can put in the .gitignore file are 

* Blank lines or lines staring with a # are ignored
* Standard glob patterns work, and will be applied recursively throughout the entire working tree.
* You can start patterns with a forward slash (/) to avoid recursivity
* You can end patterns with a forward slash (/) to specify a directory

[comment]: <> (TODO: Need some examples here)

## Git Aliases

Git has aliasing capabilities that allow you to create shorthands for commands.

`git config --global alias.co checkout` - Type `git co` instead of `git checkout`

`git config --global alias.br branch` - Type `git br` instead of `git branch`

`git config --global 