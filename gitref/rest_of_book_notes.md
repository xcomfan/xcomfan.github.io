---
layout: page
title: "Raw Notes"
permalink: /gitref/rawnotes/
---
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

## Git Internals

### Git Internals - Plumbing and Porcelain

[comment]: <> (TODO: go though with a .git directory and see all this in action)

Below is what a freshly `git init` directory looks like.

```
$ ls -F1
config
description
HEAD
hooks/
info/
objects/
refs/
```

* `description` file is used only by the GitWeb program so we don't cover it.
* `config` file contains your project-specific options
* `info` directory keeps a global exclude file for ignored patterns that you don't want to track in a .gitignore file.
* `hooks` directory contains your client or server hook scripts.

The 4 we don't see yet and will cover later are

* `index` file where Git stores your staging area information.
* `objects` directory which stores all the content for your database
* `refs` directory which stores pointers into commit objects in the objects directory (branches, tags, remotes, etc.)
* `HEAD` file which points to the branch you currently have check out

### Git Internals - Git Objects

Git is a content addressable filesystem.  This means that at its core Git is a key value store.  You can insert any content into a Git repository and Git will hand you back a unique key which you can later use to retrieve that content. 

The book goes though an example where the `git hash-object` command is used to store a file in the `objects` directory.  The key take away from this example is that if you have a file test.txt and use git hash-object to store it, each time you call git hash-object and pass it text.txt a new object is stored in the objects directory with the hash generated by the contents of the file.  In the object directory the hash is the filename for the content as it was when git hash-object is used.  The hashed stored object is called a `blob`.

#### Tree Objects

The **tree** object is using in Git internals to store the file name and allow you to store a group of files together.

In Git internals everything as stored as tree and blob objects with trees corresponding to UNIX directory entries and blobs corresponding more or less to inode or file contents.

A single tree object contains one or more entries, each of which is the SHA-1 hash of a blob or subtree with tis associated mode, type, and filename.

Git normally creates a tree by taking the state of your staging area or index and writing a series of tree objects from it.

#### Commit Objects

The tree object essentially stores a snapshot, but we don't have an easy way of referencing that snapshot (SHA-1 values are hard to remember) and we have no information about the snapshot (who saved it or when they saved it).  This is the information that the commit object stores for you.

A commit object stores the top level tree for the snapshot of the project at that point, the parent commits if any, the author/committer information, a timestamp, and after a blank line the commit message.

The stored objects, tree objects and commit objects make up the history you see when you use the `git log` command.  This is essentially what Git does when you run the git add and git commit commands — it stores blobs for the files that have changed, updates the index, writes out trees, and writes commit objects that reference the top-level trees and the commits that came immediately before them.

#### Object Storage

There is a header stored with every object you commit to your Git object database.  This header stores the type of object (blob or tree), the size in bytes of the content and a null byte at the end.

Git concatenates the header and the original content adn then calculates the SHA-1 checksum of that new content.  Git compresses the new content with zlib and writes the zlib-deflated content toa n object on disk.  The file is stored in objects directory with first 2 chars of the sha1 being the subdirectory and the remaining 38 the filename.  All Git objects are stored this way.

## Git Internals - Git References

Within the .git directory there is a refs directory.  This directory contains files which store the SHA-1 for the commits in your projects that are the HEADs for your local and remote branches.  For example the HEAD file you see in your .git directory refers to .git/refs/heads/main if you are on the main branch.

```
$ cat HEAD
ref: refs/heads/main
```

When you are in a detached head state (if you check out a tag, commit or remote branch) the value in the HEAD file will be a specific SHA-1 of a commit.

### Tags

In addition to blobs, trees and commits; tags are the fourth object type in Git.  Like the commit objects, a tag contains a tagger, a date, a message, and a pointer.  The main difference is that a tag object generally points to a commit object instead of a tree.  Its like a branch reference but it never moves.  It always points to the ame commit but gives it a friendlier name.

### Remotes

The third type of reference is the remote reference.  If you add a remote and put to it, Git stores the commit SHA-1 value you last pushed to that remote for each branch in the refs/remotes directory.

Remote references are considered read only.  You can git checkout one, but Git won't symbolically reference HEAD to one so you will never update it with a commit.  Git manages them as bookmarks of the last known state of where those branches were on those servers.

## Git Internals - Packfiles

Packfiles are the Git mechanism for organizing files in stored in your commits to save disk space.  Instead of story the copy of the file in each commit, packer files store and organize the deltas between those files.

## Git Internals - Transfer Protocols

Git can transfer data between two repositories via a dumb protocol or a smart one.  The dump protocol is used if you are setting up a read only repository to be read only over HTTP via GET requests.  The smart protocol needs Git code running on the server.  It can figure out what the client has and needs and generate the packfile for that request.

## Git Internals - Maintenance and Data Recovery

### Maintenance

Git periodically runs the git auto gc (garbage collection) call which you can trigger manually by running `git gc --auto`.  This usually does nothing as you need to have 7K loose files or more than 50 packfiles before this command takes action.  These limits can be adjusted via the `gc.auto` and `gc.autopacklimit` configuration settings.  When this call does act it will place the loose files into a packfile and consolidate the packfiles into one big packfile.  It also removes objects that are not accessible from any commit and are a few months old.

### Data Recovery

If you force delete a branch that had work in it, or you hard reset a branch abandoning the commits there you may be able to recover the work.

For example in the below scenario...

```
$ git log --pretty=oneline
ab1afef80fac8e34258ff41fc1b867c702daa24b Modify repo.rb a bit
484a59275031909e19aadb7c92262719cfcdf19a Create repo.rb
1a410efbd13591db07496601ebc7a059dd55cfe9 Third commit
cac0cab538b970a37ea1e769cbbde608743bc96d Second commit
fdf4fc3344e67ab068f836878b6c4951e3b15f3d First commit
```

If you run the command `git reset --hard 1a410efbd13591db07496601ebc7a059dd55cfe9`  You effectively loose the top two commits.  See blow...

```
$ git log --pretty=oneline
1a410efbd13591db07496601ebc7a059dd55cfe9 Third commit
cac0cab538b970a37ea1e769cbbde608743bc96d Second commit
fdf4fc3344e67ab068f836878b6c4951e3b15f3d First commit
```

One way to get your lost work back is via the `reflog` command.  

[comment]: <> (TODO: We covered reflog in earlier notes makes the link here.  I should break out the content here into a data recover section and have each scenario plus recovery steps listed.)

When you run `git reflog` it will show you the history of where you have been as in the example below.

```
$ git reflog
1a410ef HEAD@{0}: reset: moving to 1a410ef
ab1afef HEAD@{1}: commit: Modify repo.rb a bit
484a592 HEAD@{2}: commit: Create repo.rb
```

You can use the git log format on your reflog data by using the -g argument o git log.

`git log -g`

Once you identify the commit you lost you can create a new branch from that commit with the command below.  This will give you a new branch named recover-branch which will be where your master branch used to be making the lost commits reachable again.

`git branch recover-branch ab1afef`

If for some reason you don't have the lost work in your reflog history.  You can try using the git fsck utility.  

`git fsck --full`

When run with full option git fsck will show you all of the commits that are not reachable.  You can identify your lost commit from that output and create a branch from it as you did above.

[comment]: <> (TODO: Book glosses over the way you identify the commit you want to go to so you may want to run though the exercise in the book to get a better idea.)

### Removing Objects

This is useful if someone committed a large binary file to your repo. 

***This technique is destructive to the commit history***  It rewrites every commit object sine the earliest tree y ou have to modify to remove a large file reference.  If you do this when users have based their work on the commit you have to notify all users to rebase their work onto your new commits.

[comment]: <> (TODO: Should run though this exercise as well)

Kind of a side note but you can use the below command to see how much space your repo is using.

To find the large file you can use the Git plumbing command git verify-pack and sort ont he third field

`git verify-pack -v .git/objects/pack/pack-29…69.idx | sort -k 3 -n | tail -3`

Once you find the commit with the large file you can use rev-list command to find the file itself.

`git rev-list --objects --all | grep 82c99a3`

Now you need to remove this file from all trees in your past.  You can see what commits modified this file with the command...

`git log --oneline --branches -- git.tgz`

Based on our example you must rewrite all commits downstream from 7b30847.  To do this we use filter-branch command.

[comment]: <> (TODO: I have this command elsewhere in notes need to make the link.)

```
$ git filter-branch --index-filter \
  'git rm --ignore-unmatch --cached git.tgz' -- 7b30847^..
Rewrite 7b30847d080183a1ab7d18fb202473b3096e9f34 (1/2)rm 'git.tgz'
Rewrite dadf7258d699da2c8d89b09ef6670edb7d5f91b4 (2/2)
Ref 'refs/heads/master' was rewritten
```

The --index-filter option is similar to the --tree-filter option used in Rewriting History except that instead of passing a command that modifies the files checkout on disk you are modifying your staging area or index each time.

Rather than remove a specific file with `rm file` you have to remove it with git rm -- cached.  You must remove it from the index, not from disk.  The reason to do it this way is speed because Git doesn't have to check out each revision to disk before running your filter, the process can be much much faster.

The --ignore-unmatch option to git rm tells it not to error out if the pattern you’re trying to remove isn’t there. Finally, you ask filter-branch to rewrite your history only from the 7b30847 commit up, because you know that is where this problem started. Otherwise, it will start from the beginning and will unnecessarily take longer.

Your history no longer contains a reference to that file. However, your reflog and a new set of refs that Git added when you did the filter-branch under .git/refs/original still do, so you have to remove them and then repack the database. You need to get rid of anything that has a pointer to those old commits before you repack:

```
$ rm -Rf .git/refs/original
$ rm -Rf .git/logs/
$ git gc
Counting objects: 15, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (11/11), done.
Writing objects: 100% (15/15), done.
Total 15 (delta 1), reused 12 (delta 0)
```

You can see from the size value that the big object is still in your loose objects, so it’s not gone; but it won’t be transferred on a push or subsequent clone, which is what is important. If you really wanted to, you could remove the object completely by running git prune with the --expire option:

```
$ git prune --expire now
$ git count-objects -v
count: 0
size: 0
in-pack: 15
packs: 1
size-pack: 8
prune-packable: 0
garbage: 0
size-garbage: 0
```

## Git Internals - Environment Variables

Git always runs inside a bash shell and uses environment variables to determine how it behaves.  Below are the most useful environment variables.

### Global behavior environment variables

**GIT_EXEC_PATH** - determines where Git looks for its sub-programs such as git-commit, git-diff etc.  You can check the current setting with the command `git --exec-path`

**HOME** - Not usually customized as many things depend on it.  This variable is where Git looks for the global config file.

**PREFIX** - Similar to the HOME variable but for system-wide configuration file.

**GIT_CONFIG_NOSYSTEM** - if set, disabled use of the system wide configuration file.  This is useful if your system config is interfering with your commands, but you don't have access to change or remove it.

**GIT_PAGER** - controls the program used to display multi-page output on the command line.  If this is not set PAGER will be used as the fallback.

**GIT_EDITOR** - The editor Git will launch when the user needs to edit text.  In a commit for example.

### Repository location environment variables

**GIT_DIR** - The location of the .git folder.  If this directory is not specified Git will walk up directory tree till it finds or gets to ~ or / directories.

**GIT_CEILING_DIRECTORY** - Controls the behavior of searching for a .git directory.  If you have some slow loading directories such as a tape drive you may want to prevent Git from searching those directories.

**GIT_ALTERNATE_OBJECTS_DIRECTORIES** - Is a colon separated list which tells Git where to check for objects if they are not in GIT_OBJECT_DIRECTORY.  If you have a lot of projects with large files that have the exact same contents, this can be used to avoid storing too many copies of them.

### Pathspec environment variables

A "pathspec" refers to how you specify paths to things in Git, including the use of wildcards.  These are used in the .gitignore file as well as in command lines such as git add *.c.

[comment]: <> (TODO: Add the above to your definition list.)

**GIT_GLOB_PATHSPECS and GIT_NOGLOB_PATHSPECS** - control the default behavior of wildcards in pathspecs.  If GIT_GLOB_PATHSPECS is set to 1, wildcard characters act as wildcards (which is the default); if GIT_NOGLOB_PATHSPECS is set to 1, wildcard characters only match themselves, meaning something like *.c would only match a file named “\*.c”, rather than any file whose name ends with .c. You can override this in individual cases by starting the pathspec with :(glob) or :(literal), as in :(glob)\*.c.

**GIT_LITERAL_PATHSPECS** - Makes it so that no wildcard expansion works and override prefixes are disabled as well.

**GIT_ICASE_PATHSPECS** - sets all pathspecs tow ork in a case-insensitive manner.

### Committing environment variables

The final creation of a Git commit object is usually done by git-commit-tree which uses the following environment variables falling back on config values if not set.

**GIT_AUTHOR_NAME** - Human readable name in the "author" field

**GIT_AUTHOR_DATE** is the timestamp used for the “author” field.

**GIT_COMMITTER_NAME** sets the human name for the “committer” field.

**GIT_COMMITTER_EMAIL** is the email address for the “committer” field.

**GIT_COMMITTER_DATE** is used for the timestamp in the “committer” field.

**EMAIL** is the fallback email address in case the user.email configuration value isn’t set. If this isn’t set, Git falls back to the system user and host names.

### Networking environment variables

**GIT_CURL_VERBOSE** - Git uses curl library to do network operations over HTTP.  This setting tells Git to emit all the messages generated by that library similar to `curl -v`

**GIT_SSL_NO_VERIFY** - Tells Git not to verify SSL certificates.

### Diffing and merging environment variables

**GIT_DIFF_OPTS** - The only valid values are `-u<n>` or `--unified=<n>`, which controls the number of context lines shown in a git diff command.

**GIT_MERGE_VERBOSITY** - controls the output for the recursive merge strategy. The allowed values are as follows:
* 0 outputs nothing, except possibly a single error message.
* 1 shows only conflicts.
* 2 also shows file changes.
* 3 shows when files are skipped because they haven’t changed.
* 4 shows all paths as they are processed.
* 5 and above show detailed debugging information.

The default value is 2