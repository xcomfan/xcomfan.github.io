---
layout: page
title: "Git Configuration"
permalink: /gitref/config/
---

[comment]: <> (TODO: REV MARKER)

[comment]: <> (TODO: TODO: Write up how to make the git log act in a custom way without having to specify arguments each time.  I recall that was possible via git configuration files.)

## Git Configuration

### Customizing Git - Git Configuration

#### Git Configuration

This section was just a review of the 3 config levels I already have in notes.

##### Basic Client Configuration

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

##### Colors in Git

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

##### External Merge and Diff Tools

This section shows how to set up the Perforce merge tool (a free graphical merge-conflict tool) instead of the Git default merge tool.  The example here should work on Mac and Linux.  You will need to adjust the paths used for Windows.

You can download the P4Merge tool [here](https://www.perforce.com/product/components/perforce-visual-merge-and-diff-tools)

First create a merge wrapper script named `extMerge` as below.  Adjust the path to your binary.

```
#!/bin/sh
/Applications/p4merge.app/Contents/MacOS/p4merge $*
```

Next create the `extDiff` diff wrapper script as below.  This wrapper checks to make sure seven arguments are provided and passes two of them to your merge script.  By default git passes the following arguments to the diff program.

path, old-file, old-hex, old-mode, new-file, new-hex, new-mode

In our case we only want the old-file and new-file arguments so we have the wrapper script pass those.

```
#!/bin/sh
[ $# -eq 7 ] && /usr/local/bin/extMerge "$2" "$5"
```

Make sure to give execute permission to both scripts.

Now we can set up our Git config to use our custom merge resolution and diff tools.

```
git config --global merge.tool extMerge
git config --global mergetool.extMerge.cmd \
  'extMerge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"'
git config --global mergetool.extMerge.trustExitCode false
git config --global diff.external extDiff
```

alternatively you can edit your ~/.gitconfig with the following changes

```
[merge]
  tool = extMerge
[mergetool "extMerge"]
  cmd = extMerge "$BASE" "$LOCAL" "$REMOTE" "$MERGED"
  trustExitCode = false
[diff]
  external = extDiff
```

With this changes when you run `git diff 32d1786b1^ 32d1786b1` instead of the Git output, Git will fire up P4Merge.  If you try to merge two branches and have a conflict.  Running `git mergetool` will now start P4Merge to let you resolve the conflicts through that GUI tool.

Git comes preset to use a number of other merge-resolution tools without your having to set up the cmd configuration.  To see a list of supported tools run the below command.

`git mergetool --tool-help`

If for example you are not interested in using KDiff3 for diff, but rather want to use it just for merge resolution, and the kdiff3 command is in your PATH, then you can run the command below.

`git config --global merge.tool kdiff3`

##### Formatting and Whitespace

**core.autocrlf**

If you are working on Windows with people that are not you will probably run into line ending issues at some point.  Windows uses both carriage-return and linefeed characters for newlines while MacOS and Linux only use linefeed.  Many editors on Windows replace or append LF-style endings with CRLF.

Git can handle this by auto-converting CRLF line endings into LF when you add a file to the index and vice versa when it checks out code onto your filesystem.  You can turn this on with the `core.autocrlf` setting.

If you are on a Windows machine, set it to `true` - this will convert LF endings into CRLF when you check out code.

`git config --global core.autocrlf true`

if you're on a Linux or MacOS system that uses LF line endings, you don't want Git to automatically convert them when you check out files, however if a file with CRLF endings accidentally gets introduced, then you may want Git to fix it.  You can tell Git to convert CRLF to LF on commit but not the other way around by setting `core.autocrlf` to `input`.  This setup should leave you with CRLF endings in Windows checkout, but LF endings on macOS and Linux systems and in the repository.

`git config --global core.autocrlf input`

If you are a Windows programmer doing a Windows only project, then you can turn off this functionality, recording the carriage returns in the repository by setting `core.autocrlf` to `false`

`git config --global core.autocrlf false`

**core.whitespace**

Git has the ability to fix 6 types of whitespace issue.  3 are enabled by default and 3 are disabled by default.

3 that are on by default are

* blank-at-eol - looks for spaces at end of line
* blank-at-eof - looks for spaces at end of a file
* space-before-tab - looks for spaces before tabs at the beginning of a line

3 that are disabled by default are

* indent-with-non-tab - looks for lines tha begin with spaces instead of tabs (controlled by the `tabwidth` option)
* tab-in-indent - watches for tabs in the indentation portion of a line
* cr-at-eol - tells Git that carriage returns at the end of lines are OK.

You can tell Git which of these you want enabled by setting `core.whitespace` to the values you want on or off.  

```
git config --global core.whitespace \
    trailing-space,-space-before-tab,indent-with-non-tab,tab-in-indent,cr-at-eol
```

Git will detect these issues when you run `git diff` command and try to color them so that you can fix them before you commit.  It will also use these values when you apply patches with `git apply`  When you are applying patches you can ask Git to warn you if it's applying patches with the specified whitespace issues:

`git apply --whitespace=warn <path>`

or you can have Git automatically fix the issue before applying the patch.

`git apply --whitespace=fix <path>`

These options apply to `git rebase` as well.  If you committed whitespace issues, but haven't pushed them upstream you can run `git rebase --whitespace=fix` to have Git automatically fix whitespace issues as it's rewriting the patches. 

##### Server Configuration

[comment]: <> (TODO: I am not 100% clear on where you run these.  I believe its on the git server if you have that configured.)

**receive.fsckObjects**

Git has capability to make sure every object received during a push matches its SHA-1 checksum and points to valid objects.  This is disabled by default because it is an expensive operation especially on large repositories.  You an use `receive.fsckObjects` to enable this functionality.

`git config --system receive.fsckObjects true`

**receive.denyNonFastForwards**

This setting lets you prevent user from pushing changes that rewrite history by using the -f flag.  This is generally good policy.  ***Note:*** There is a way to do this with hooks that let you allow the force commit to a certain set of users.

`git config --system receive.denyNonFastForwards true`

**receive.denyDeletes**

This setting will deny the deletion of branches.  If this setting is enabled to remove branches you would need to remove the ref files on the server manually.  There is a way to configure this on a per user basis via access control lists (ACLs)

`git config --system receive.denyDeletes true`

### Git Attributes

Certain settings can be specified for a specific path so that Git only applies those settings for a subdirectory of files.  These path specific settings are called Git attributes, and they can be set either in a `.gitattributes` file in one of you directories (normally the root of your project) or in the `.git/info/attributes` file if you don't want the attributes committed to your project.

You can use Git attributes to specify separate merge strategies for individual files or directories, tell Git how to diff non-text files, or have Git filter content before you check it into or out of Git.

[comment]: <> (TODO: Make a definition entry for Git attributes based on above)

#### Binary Files

You can use Git attributes to tell Git which files are binary in case it may not be able to figure it out, and how to handle those files.

##### Identifying Binary Files

Some files such as lightweight database are written as UTF-8, but area not meant to be treated as text.  You can't effectively diff or merge this kind of file.  To tell Git to treat a certain file extension as binary at an entry to your `.gitattributes` file similar to the one below.  By doing this Git won't try to convert or fix CRLF issues nor will it try to compute and display changes for these files when you run `git show` or `git diff`

`.myextension binary`

##### Diffing Binary Files

You can use Git attributes to effectively diff binary file by telling Git how to convert binary files to text format that can be compared with a diff.

###### Word documents

You can use this to let Git diff word documents.  To do this make an entry similar to the one below in your `.gitattributes` file.  You will also need to set up the "word" filter.

`.docx diff=word`

You will aso need to install the [doc2txt program](https://sourceforge.net/projects/docx2txt) to convert Word documents into text files.  You will need the below wrapper script, and Git config entry to make Git use the doc2txt program.

```
#!/bin/bash
docx2txt.pl "$1" -
```

`git config diff.word.textconv docx2txt`

###### Images

You can run images though a program that extracts their EXIF information (metadata that is recorded with most image formats).  You can do this by installing the `exiftool` which you can use to convert you images into text about the metadata.

Add the following to your `.gitattributes` file.

`*.png diff=exif`

And configure Git to use this tool

`git config diff.exif.textconv exiftool`

Not if you replace an image in your project git diff will show you the metadata changes.

#### Clean and smudge filters

[comment]: <> (TODO: Need to try this out and update note accordingly.)

Git has functionality that lets you set up a filter that runs when you check out a code to "smudge" it and when you stage it to "clean it".  For example if you want to run the `indent` program on `c` files when you check them out and the `cat` program (does nothing but we are using it as an example) when you stage the files you would add the below line to your `.gitattributes` file.

`*.c filter=indent`

And se the following Git configuration for smudge and clean.

```
git config --global filter.indent.clean indent
git config --global filter.indent.smudge cat
```

#### Exporting your repository

Git attributes allow you to do some interesting things when exporting an archive of your project

##### export-ignore

You can tell Git not to export certain files or directories when generating an archive.  For example if you don't want file sin the test directory to be exported when creating an archive you would add the following to your `.gitattributes` file and those files would not be included when you run `git archive`

`test/ export-ignore`

##### export-subset

When exporting files for deployment you can apply formatting and keyword expansion from `git log` processing to selected portions of files marked with the `export-subset` attributes.

For example if you want to include a file named LAST_COMMIT in your project, and have metadata about the last commit automatically injected into it when `git archive` runs you would add the below to your `.gitattributes` files.

`LAST_COMMIT export-subst`

and create the LAST_COMMIT template.

```
$ echo 'Last commit date: $Format:%cd by %aN$' > LAST_COMMIT
$ git add LAST_COMMIT .gitattributes
$ git commit -am 'adding LAST_COMMIT file for archives'
```

Now when you run `git archive` the contents of the archived file would show.
```
$ git archive HEAD | tar xCf ../deployment-testing -
$ cat ../deployment-testing/LAST_COMMIT
Last commit date: Tue Apr 21 08:38:48 2009 -0700 by Scott Chacon
```

#### Merge strategies

You can use Git attributes to tell Git to use different merge strategies for specific files in your project.  One useful option is to tell Git to not try to merge specific files when they have conflicts, but rather to use your side of the merge over someone else's   For example if you have if you have a database.xml file and you want your settings to be preserved on a merge.  You would add the following to your `.gitattributes` file.

`database.xml merge=ours`

And then define a dummy ours merge strategy with

`$ git config --global merge.ours.driver true`

Now if you merge in another branch instead of having merge conflicts for database.xml you will see something like the below. and database.xml stays at whatever version you originally had it at.

```
$ git merge topic
Auto-merging database.xml
Merge made by recursive.
```

### Git Hooks

Git Hooks is a way to fire off custom scripts when certain actions occur.  There are client-side and server-side Git hooks.  Client side hooks are triggered by operations such as committing and merging while server side hooks run on network operations such as receiving pushed commits.

[comment]: <> (TODO: Need to experiment with these.  I am also not sure if these get source managed or not in your project.  Need to experiment with that as well.)

#### Installing a hook

Hooks are installed in the `hooks` subdirectory of the Git directory.  In most projects that will git `.git/hooks`.  When you initialize a new project with `git init`, Git populates the hooks directory with a bunch of example scripts.  If you want to use the bundled in scripts you will need to rename them to remove the `.sample` extension.

For a hook script to be enabled, it needs bo be in the hooks directory, appropriately named and have execute permission.  You are free to use any scripting you want to write the scripts.  Use the sample scripts as a reference for naming and what arguments scripts are expecting.

#### Client-side hooks

##### Committing-Workflow Hooks

**pre-commit** - This hook runs before you get prompted for a commit message.  It is used to inspect the snapshot that is about to be committed.  Exiting non zero from this hook aborts the commit.  This is a good place to run linters, check for trailing white spaces etc.  You can skip the execution of this hook if you use the --no-verify argument to git commit (`git commit --no-verify)

**prepare-commit-msg** - This hook runs before the commit message editor is fired up, but after the default message is created.  It lets you edit the default message before the commit author sees it.  This hook is not generally useful for normal commits; rather it's good for commits where the default message is auto generated, such as templated commit messages, merge commits, squashed commits, and amended commits.  You can use it in combination with a commit template to programmatically insert information.

**commit-msg** - This hook is useful for validating the content of a commit message.  The script for this hook gets a single argument that is the path to the temporary file with the commit message written by the author.  If this script exits non zero than the commit is aborted.

**post-commit** - This script does not receive any arguments, but you can get the last commit by running `git log -1 HEAD`.  It is generally used for notifications.

##### Other Client Hooks

**pre-rebase** - This hook runs before you rebase, and can halt the process by exiting non-zero.  You can use this hook to block rebasing any commits tha have already been pushed.  The example pre-rebase hood provided by Git does this.

**post-rewrite** - This hook is run by commands that replace commits, such as `git commit --amend` and `git rebase`.  Its single argument is the command that triggered the rewrite, and it gets the list of rewrites on stdin.  This hook has many of the same uses as the post-checkout and post-merge hooks.

**post-checkout** - This hook runs after you run a successful `git checkout`.  You can use this hook to set up your working directory properly for your project environment.  Examples are moving in large binary files that you don't want source managed or auto generating documentation.

**post-merge** - This hook runs after a successful merge command.  You can use it to restore data in the working that that Git can't truck such as permissions data.

**pre-push** - This hook runs during git push, after the remote refs have been updated but before any objects have been transferred.  It receives the name and location of the remote as parameters, and a list of to be updated refs through stdin.  You can use this hook to validate a set of ref updates before a push occurs.  A non zero exit code in this hook will abort the push.

[comment]: <> (TODO: Probably need to drip this one as I don't see the use, but will keep till I read the Git internals chapter)
**pre-auto-gc** - This hook is invoked just before Git does garbage collection (by calling `git gc --auto`).  You an use it to abort the garbage collection.

#### Server-Side Hooks

[comment]: <> (TODO: I wonder if GitHub has ability for you to run these)
Server side hooks can be used to enforce project policy.  These scripts run before and after pushes to the server.  The pre hooks can exit with a non zero to reject a push and print an error message back to the client.

**pre-receive** - This hook gets a list of references being pushed on stdin.  If it exists non zero, none of the references are accepted.  You can use this hook to make sure none of the updated references are non-fast-forwards, or to do access control for all the refs and files they're modifying with the push.

**update** - This hook is similar to pre-receive, but it is run once for each branch the push is trying to update.  This script takes 3 arguments: the name of the reference (branch), the SHA-1 that reference pointed to before the push, and the SHA-1 the uer is trying to push.  If the update script exits non-zero, only that reference is rejected.
[comment]: <> (TODO: How do you push to multiple branches?)

**post-receive** - This hook runs after the entire process is completed, and can be used to update other services or notify users.  It takes the same stdin data s pre-receive hook.  Uses for this hook include emailing a list, notifying a continuous integration server, or updating a ticket tracking system.  You can even parse the commit message to see if any ticket need to be opened, modified or closed.  This script can't stop the push process, but the client doesn't disconnect until it has completed so its not a good place for long running actions.

### Customizing Git - An example Git enforced policy

The Git Pro book has an example of using hooks to set up a policy that is a good reference.  It can be found [here](https://git-scm.com/book/en/v2/Customizing-Git-An-Example-Git-Enforced-Policy)