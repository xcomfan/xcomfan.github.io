---
layout: page
title: "Linux Reference"
permalink: /linux/
---

[comment]: <> (TODO: Work your way through this page moving things into their own sections and linking out. We should get down to about a page)

## Linux standards

**POSIX** stands for portable operating system interface and it refers to a group of standards developed under IEEE.  The goal of the standard is to promote application portability at the source code level.  POSIX documents an API for a set of services that should be made available to a program by a conforming operating system.  POSIX xpecifies the API, but not the implementation.

**SUSvX** stands for single UNIX Specification Version X and it is essentially POSIX 1003.1-2001 which replaces earlier POSIX standards.

## Tasks performed by the kernel

### Process scheduling

Linux is a preemptive multitasking operating system.  Multitasking means that multiple processes can simultaneously reside in memory and each may receive use of the CPU(s). Preemptive means that rules governing which processes receive use of the CPU and for how long are determined by the kernel process scheduler (rather than by the processes themselves).

### Memory management

Memory is a limited resource that the kernel needs to manage in an equitable and efficient fashion. Linux uses virtual memory management which has the following benefits.

* Processes are isolated from each other and from the kernel so that one process can't read or modify memory of another.  
* Only part of a process needs to be kept in memory.  Allowing more processes to run in the RAM available.

### Provision of a file system

The kernel provides a file system on disk, allowing files to be created, retrieved, updated, deleted, and so on.

### Creation and termination of processes

The kernel can load a new program into memory and provide it with resources (CPU, RAM, access to files). Once the program has completed the kernel ensures that the resources it used are freed for subsequent programs.

### Access to devices

The kernel provides an interface that that simplifies access to devices such as mice, keyboards, disks etc.

### Networking

The kernel transmits and receives network messages (packets) on behalf of user processes.

### Provision of a system call application programming interface (API)

The API interface that lets you build programs that utilize the services provided by the kernel

## Kernel Mode and User Mode

Modern processors operate in at least two modes.  user mode and kernel mode.  Hardware instructions allow switching from one mode to the other.  

Areas of virtual memory can be marked as being part of user space or kernel space.  When running in user mode the CPU cannot access kernel space memory.  In kernel mode the CPU can access both user space and kernel space.   Attempts to access kernel space in user mode will result in a hardware exception.

Certain operations can only be performed when processor is in kernel mode.  Some examples are…

* heart instruction
* accessing the memory management hardware
* initiating device I/O operation

## Process versus kernel view of the system

### Process view

Many things happen transparently for a process. A process doesn't know where it is located in RAM or even if its in RAM or a swap area. It does not know where on disk the files its accessing are, and can't communicate directly with I/O. Process operates in isolation.  It can't directly communicate with another process, create a new process or even end its own existence.

### Kernel view

Kernel facilitates running of all processes on the system, and kernel decides which process will obtain access to the CPU next and for how long.  Kernel maintains data structures with information about all running processes and updates these structures as processes are created, change state and terminate. kernel maintains low level data structures that map the virtual memory of each process into the physical memory of the computer and the swap areas on disk. All communication between processes is done via mechanisms provided by the kernel.  In response to requests form processes kernel creates and terminates processes.
Kernel (in particular device drivers) performs all direct communication with input and output devices transferring information to and from user processes as required.

## Shell

A shell is a special purpose program designed to read commands typed by a user and execute appropriate programs in response to those commands.  The term login shell is used to denote the process that is created to run a shell when the user first logs in.

### Directory hierarchy, links and files

#### File Types

Within the file system each file is marked with a type

* regular or plain files
* directory - special file whose contents take the form of a table of filenames coupled with references to the corresponding files (links) Every directory contains at least two entries . (dot) and .. (dot-dot) which are links to current and parent directory. In the / root directory the . (dot) refers to itself.
* symbolic link - provides an alternative name for a file. A normal link is a filename plus pointer entry in a directory, a symbolic link is a specially marked file containing the name of another file.

#### Current Working Directory

Each process has a current working directory (sometimes called the process working directory). A process inherits its current working directory from its parent process.

## File I/O Model

Everything in Linux is a file and the same API calls (open(), read(), write(), close()) are used to perform I/O on all types of files including devices.  Kernel translates the applications I/O requests into appropriate file-system or device driver operations that perform I/O on the target file or device.  Kernel essentially provides one file type: a sequential stream of bytes (which for disk files can be randomly accessed using lseek() system call.

### File descriptors

The I/O system calls refer to open files using a file descriptor (usually a small non negative integer).  Normally a process inherits 3 file descriptors when it is started by a shell.  

* descriptor 0 is standard input, the file from which the process takes its input.
* descriptor 1 is standard output, the file to which the process writes its output.
* descriptor 2 is standard error, the file to which the process writes error messages and notification of exceptional or abnormal conditions.

In an interactive shell or program these three descriptors are normally connected to the terminal.  In the stdio library they correspond to stdin, stdout, and stderr.  stdio library is the standard C library that provides fopen(), fclose(), scanf(), printf() etc…

## Processes

A process is an instance of an executing program.
        
### Process memory layout

A process is logically divided into the following parts, known as segments

* Text - The instructions of the program
* Data - The static variables used by the program
* Heap - An area from which programs can dynamically allocate extra memory
* Stack - A piece of memory that grows and shrinks as functions are called and return and that is used to allocate storage for local variables and function call linkage information.

### Process creation and program execution

A process can create a new process using the fork() system call. The process that calls fork() is referred to as the parent process and the new process is called the child process. The kernel creates the child process by making a duplicate of the parent process.  The child inherits copies of the parent's data, stack and heap segments which it may then modify independently of the parent's copies. The program text, which is placed in memory marked as read-only, is shared by the two processes. The child process goes on to either execute a different set of functions in the same code as the parent, or frequently to use the execve() system call to load and execute an entirely new program. An execve() call destroys the existing text, data, stack and heap segments, replacing them with new segments based on the code of the new program.

Each process has a unique integer process identifier PID and a parent process identifier PPID which identifies the process that requested the kernel to create this process.

### Process termination and termination Status

A process can terminate in one of two ways

* By requesting its own termination use the _exit() system call or related exit() library funciton.
* By being killed by the delivery of a signal.

In either case the process yields a termination status. A small nonnegative integer value that is available for inspection by the parent process using the wait() system call. In the case of a call to _exit() the process explicitly specifies its own termination status. If a processes is killed by a signal the termination status is set based on the type of signal. By convention termination status of 0 indicates that the process succeeded, and a nonzero status indicates that some error occurred. Most shells make the termination status available in the $? variable.

### Process user and group identifiers

Each process has a number of associated user IDs (UIDs) and group IDs (GIDs). Real user ID and real group ID: These identify the user and group to which the process belongs. A new process inherits these IDs from its parent. A login shell gets its real user ID and real group ID from the corresponding fields in the sytem password file

Effective user ID and effective group ID: These in conjunction with the supplementary group IDs discussed below are used in determining the permissions that the process has when accessing protected resources such as files and inter-process communication objects. Typically the effective IDs are the same as the real ones. Changing the effective IDs is a mechanism that allows a process to assume the privileges of another user or group as described in a moment.

Supplementary group IDs: These IDs identify additional groups to which a process belongs. A new process inherits its supplementary group IDs from its parent. A login shell gets its supplementary group IDs from the system group file

### The init process

When booting the system the kernel creates a special process called init the parent of all processes which is derived from the program file /sbin/init. All processes on the system are created using fork() either by init or one of its decedents. Init process always has the PID of 1 and runs with superuser privileges. Init process cannot be killed even by the superuser and terminates only when system is shut down. The main task of init is to create and monitor a range of processes required by a running system. For details see the init(8) man page.

### Resource limits

A process can establish upper limits on its consumption of various resources using the setrlimit() system call. Each resource limit has two associated values.

* A soft limit which limits the amount of resource the process may consume.
* A hard limit which is a ceiling on the value to which the soft limit may be set.

An unprivileged process may change its soft limit for a particular resource to any value up to the hard limit, but can only lower its hard limit. Resource limits of the shell can be adjusted using the ulimit command.  These limits are inherited by the child processes that the shell creates to execute commands.
