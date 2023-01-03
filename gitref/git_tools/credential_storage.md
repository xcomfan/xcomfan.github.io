---
layout: page
title: "Bundling"
permalink: /gitref/git_tools/credential_storage
---

[comment]: <> (TODO: Confirm that the way you broke this out is correct for the different OSes)

## General credential options

Git has built in credential storage with the following 3 modes.

* The default mode is not to cache at all and prompt for a username and password with every connection.
* Cache mode will keep credentials in memory, but not on disk for a certain period of time. 
* The store mode saves the credential in a plain-text file on disk, and they never expire.  The file is stored in users home directory.

You can choose between these methods by setting a Git configuration value

`git config --global credential.helper cache`

You can provide additional options for example the store mode has a --file option for specifying file location to store credentials in.  The cache mode has a --timeout option.

`git config --global credential.helper ``store --file ~/.my-credentials`` `

You can configure multiple helpers to be used.  When looking for credentials for a particular host Git will query them in order and stop after the first answer is provided.

The git-credential-store is a separate binary.  You are able to replace it with any credential helper you choose.

[comment]: <> (TODO: Book has a script for doing this that you should look into.)

## Mac

Git comes with an oskeychain mode which caches credentials in the secure keychain that's attached to your system account.

[comment]: <> (TODO: How do you use this?)

## Windows

You can enable Git credential manager feature when installing Git for Windows (can be installed separately as well).  This will use Windows credential store to control sensitive information.