---
layout: page
title: "Linux Universal File I/O Model"
permalink: /linux/universal_io_model
---

## Universality of I/O

Everything in Linux is a file and the same API calls (open(), read(), write(), close()) are used to perform I/O on all types of files including devices. The kernel translates the applications I/O requests into appropriate file-system or device driver operations that perform I/O on the target file or device. Th kernel essentially provides one file type: a sequential stream of bytes. Universality of I/O means that the I/O access API (read(), write(), close() etc.) will work for any type of file. This is achieved by ensuring that each file system and device driver implements the same set of I/O system calls.

## File descriptors

All systems performing I/O refer to open files using a **file descriptor** a (usually small) non negative integer. There can be multiple descriptors pointing to the same file. These descriptors may belong to one or multiple processes.

A **file offset** is shared between these open descriptors so if one updates the offset the other descriptor sees that change.

**File descriptor** flags (such as close on exec flag) are exclusive to each file descriptor.

By convention most programs expect to use the three standard file descriptors that are opened by the shell on the programs behalf.

| File Descriptor | Purpose | POSIX name | stdio stream | Description |
| --------------- | ------- | ---------- | ------------ | ----------- |
| 0 | standard input | STDIN_FILENO | stdin | the file from which the process takes its input |
| 1 | standard output | STDOUT_FILENO | stdout | the file to which the process writes its output |
| 2 | standard error | STDERRR_FILENO | stderr | the file to which the process writes error messages and notification of exceptional or abnormal conditions |

The kernel maintains a system wide table of all open file descriptors.  Each descriptor in the table has the following.

* current file offset
* status flags specified when opening file
* file access mode
* settings related to signal driven I/O
* a reference to the i-node object of this file

In an interactive shell or program these three descriptors are normally connected to the terminal.  In the stdio library they correspond to stdin, stdout, and stderr.  stdio library is the standard C library that provides fopen(), fclose(), scanf(), printf() etc…

## I/O Redirection

`$./myscript > results.log 2>&1`

The 2>&1 informs the shell that we wish to have standard error (file descriptor 2) redirected to the same place to which standard output (file descriptor 1) is being sent.

## The /dev/fd Directory

You can see open file descriptors under `/dev/fd/n` where `n` is a number corresponding to one of the open file descriptors.

## Directory hierarchy, links and files

### File Types

Within the file system each file is marked with a type

* regular or plain files
* directory - special file whose contents take the form of a table of filenames coupled with references to the corresponding files (links) Every directory contains at least two entries . (dot) and .. (dot-dot) which are links to current and parent directory. In the / root directory the . (dot) refers to itself.
* symbolic link - provides an alternative name for a file. A normal link is a filename plus pointer entry in a directory, a symbolic link is a specially marked file containing the name of another file.

### Current Working Directory

Each process has a current working directory (sometimes called the process working directory). A process inherits its current working directory from its parent process.
