---
layout: page
title: "Process Scheduling and Management"
permalink: /linux/processes
---

[comment]: <> (TODO: There is scattered mention of /proc in this section should review and make dedicated section for it.)

[comment]: <> (TODO: Should do a read and reorganize pass of this section.)

## Processes and Programs

A **process** is an instance of an executing program. A **program**  is a file containing a range of information that describes how to construct a process at run time. This information includes the following. One program can be used to construct many processes.

* Binary format identification - describes the format of the executable file.  In modern systems it is usually ELF (Executable and Linking Format)

* Machine language instructions

* Program entry-point address

* Data values used to initialize variables and literal constants (strings) used by the program.

* Symbol and relocation tables which describe the location and names of functions and variables within the program. Used for debugging and run time symbol resolution.

* Shared library and dynamic linking information

From the kernel point of view a process consists of user-space memory containing program code and variables used by that code, and a range of kernel data structures which maintain information about the state of the process.

### Process memory layout

A process is logically divided into the following parts, known as segments

* Text - The instructions of the program
* Data - The static variables used by the program
* Heap - An area from which programs can dynamically allocate extra memory
* Stack - A piece of memory that grows and shrinks as functions are called and return and that is used to allocate storage for local variables and function call linkage information.

## Process scheduling

Linux is a preemptive multitasking operating system.  Multitasking means that multiple processes can simultaneously reside in memory and each may receive use of the CPU(s). Preemptive means that rules governing which processes receive use of the CPU and for how long are determined by the kernel process scheduler (rather than by the processes themselves).

## Process creation and program execution

A process can create a new process using the `fork()` system call. The process that calls fork() is referred to as the parent process and the new process is called the child process. The kernel creates the child process by making a duplicate of the parent process. The child inherits copies of the parent's data, stack and heap segments which it may then modify independently of the parent's copies. The program text, which is placed in memory marked as read-only, is shared by the two processes. The child process goes on to either execute a different set of functions in the same code as the parent, or frequently to use the `execve()` system call to load and execute an entirely new program. `An execve()` call destroys the existing text, data, stack and heap segments, replacing them with new segments based on the code of the new program.

## Process termination and termination Status

A process can terminate in one of two ways

* By requesting its own termination using the `_exit()` system call or related `exit()` library function.
* By being killed by the delivery of a signal.

In either case the process yields a termination status. A small nonnegative integer value that is available for inspection by the parent process using the `wait()` system call. In the case of a call to `_exit()` the process explicitly specifies its own termination status. If a processes is killed by a signal the termination status is set based on the type of signal.

By convention termination status of 0 indicates that the process succeeded, and a nonzero status indicates that some error occurred. Most shells make the termination status available in the `$?` variable.

### The init process

When booting the system the kernel creates a special process called `init` the parent of all processes which is derived from the program file `/sbin/init`. All processes on the system are created using `fork()` either by `init` or one of its decedents. Init process always has the `PID` of `1` and runs with superuser privileges. Init process cannot be killed even by the superuser and terminates only when system is shut down. The main task of `init` is to create and monitor a range of processes required by a running system. For details see the init(8) man page.

## Process ID and Parent Process ID

* The parent process ID of any process can be found in the `/proc/PID/status` file
* The process family tree can be seen by calling the `pstree` command.

## Command Line Arguments

Every C program must have a function called main() which is the point where execution of the program starts.

Command line arguments (the separate words parsed by the shell) are made available via two arguments to the function main()

* The first `argc` indicates how many arguments we have
* The second `argv` is an array of pointers to the command-line arguments.
* `argv[0]` will be the name of the program itself

The command line arguments of any process can be found in `/proc/PID/cmdline` file.  A program can access its own command line arguments via `/proc/self/cmdline`.

## Environment List

A program will inherit the environment list from parent.  This can be used as a very basic version of inter process communication. Within a C program, the environment list can be accessed using the global variable `char **environ`

## Process Credentials

[comment]: <> (TODO: For now lets keep here, but need to create a deep dive intor permissions and link to that eventually)

Every process has a set of associated numeric user identifiers (UIDs) and group identifiers (GIDs).  Sometimes, these are referred to as process credentials. These credentials are as follows.

* real user ID and group ID
  * Identify the user and group to which the process belongs.
  * As part of login process a login shell gets its real user and group IDs from the third and fourth fields of the user's password record in `/etc/passwd` file.
  * When a new process is created (when shell executes a program), it inherits these identifiers from its parent.

* effective user ID and group ID
  * Normally, the effective user and group IDs have the same values as the corresponding real IDs, but there are two ways in which the effective IDs can assume different values.
    * via system calls TBD
    * via execution of set-user-ID and set-group-ID programs

* saved set-user-ID and saved set-group-ID
  * A set-userID program allows a process to gain privileges it would not normally have, by setting the process's effective user ID to the same value as the user ID (owner) of the executable file.
  * A set-groupID program tasks with the process effective group ID.
  * These are set with the `chmod +s` command
  * When these programs are run the kernel sets the effective user or group ID accordingly.
  * This allows you to grant permissions to a process it would not normally have. For example is program is owned by root and has the suid set, then when you run that program you have root permissions. This works for all users not just root.

* file system user ID and group ID
  * supplementary group IDs
    * These are the additional groups that a user belongs to and the permissions that are inherited by processes based on that.

The credentials of any process can be found by examining the Uid, Gid and Groups lines provided in the `/proc/PID/status` file. The Uid and Gid lines list the identifiers in the order real, effective, saved set and file system.

## Resource limits

A process can establish upper limits on its consumption of various resources using the `setrlimit()` system call. Each resource limit has two associated values.

* A soft limit which limits the amount of resource the process may consume.
* A hard limit which is a ceiling on the value to which the soft limit may be set.

An unprivileged process may change its soft limit for a particular resource to any value up to the hard limit, but can only lower its hard limit. Resource limits of the shell can be adjusted using the `ulimit` command.  These limits are inherited by the child processes that the shell creates to execute commands.

## Inter-process Communication and Synchronization

Linux supports the following methods for processes to communicate with each other

* signals - used to indicate that an event has occurred
* pipes - FIFOs which can be used to transfer data between processes directly related to the shell `|` concept
* sockets - can be used to transfer data from one file to another either on the same computer or over a network.
* file locking - allows a process to lock a region of a file in order to prevent other processes from reading or updating the file contents
* message queues - used to exchange messages (packets of data) between processes.
* semaphores - used to synchronize the actions of processes
* shared memory - allows two or more processes to share a piece of memory.  When one process makes a change in memory the other processes instantly see it.

### Signals

Signals are often called software interrupts. The arrival of a signal informs a process that some event or exceptional condition has occurred. Each signal type is identified by a different integer defined with symbolic names of the form `SIGxxxx`. Signals can be sent to a process by the kernel, by another process (with suitable permission) or by the process itself.

The kernel can send a signal to a process for any of the following reasons.

* User typed the interrupt character (usually Control-C)
* one of the processes children has terminated
* a timer (alarm clock) set by the process has expired
* the process attempted to access an invalid memory address.
* Within the shell the kill command and the kill() system call can be used to send a signal to a process.

When a process receives a signal it takes one of the following actions depending on the signal.

* it can ignore the signal
* it can be killed by the signal
* it can be suspended until later being resumed by receipt of a special-purpose signal.
* It can run a signal handler to act on the signal.

signal handler is a function that is automatically invoked when a signal is called.

The interval between the time a signal is generated and the time it is delivered the signal is said to be pending for a process. Normally a pending signal is delivered as soon as the receiving process is scheduled to run or immediately if the process is already running. It is also possible to block a signal by adding it to the processes signal mask. If a signal is generated while it is blocked, it remains pending until it is later unblocked.(removed from signal mask)

## Threads

Each process can have multiple threads of execution. Threads can be though of as processes that share the same virtual memory as well as other attributes. Each thread executes the same program code and shares the same data area and heap, however each thread has its own stack containing local variables and function call linkage information. Threads can communicate with each other via the global variables they share. The threading API provides condition variables and mutexes which are primitives that enable threads of a process to communicate and synchronize their actions. Threads can also communicate with each other using the IPC and synchronization mechanism.

## Process Groups and Shell Job Control

Each program executed by the shell is started in a new process. All major shells except the Bourne shell provide an interactive feature called job control which allows a user to simultaneously execute and manipulate multiple commands or pipelines. In job control shells, all of the processes in a pipeline are placed in a new process group or job. Each process in a process group has the same integer process group identifier which is the same as one of the process IDs of one of the processes in the group.  This process is called the process group leader. Kernel allows for signals to be delivered to on all members of a process group.  Job control shells use this feature to allow the user to suspend or resume all of the processes in a pipeline.

## Sessions, Controlling Terminals, and Controlling Processes

A session is a collection of process groups (jobs).  All of the processes in a session have the same session identifier. A session leader is the process that created the session, and its process ID becomes the session ID. Sessions are used mainly by job control shells. All of the process groups created by a job-control shell belong to the same session as the shell, which is the session leader. Sessions usually have an associated controlling terminal. The controlling terminal is established when the session leader process first opens a terminal device. For a session created by an interactive shell, this is the terminal at which the user logged in. A terminal may be the controlling terminal of at most one session. As a consequence of opening the controlling terminal, the session leader becomes the controlling process for the terminal. The controlling process receives a SIGHUP signal if a terminal disconnect occurs. At any point in time, one process group in a session is the foreground process group (foreground job) which may read input from the terminal and send output to it. If the user types the interrupt character (usually Control-C) or the suspend character (usually Control-Z) on the controlling terminal, then the terminal driver sends a signal that kills or suspends (stops) the foreground process group. A session can have any number of background process groups (backend jobs), which are created by terminating a command with the ampersand (&) character.
