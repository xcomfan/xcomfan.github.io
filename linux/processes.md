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

## Process Information

### The /proc File System

`/proc` is a virtual file system that provides an interface to kernel data structures in a form that looks like files and directories. `/proc/PID` allows you to view information about each process running on the system. In most cases a process must be privileged to write to the `/proc` directory.

The `init` process has a PID of 1 and details for it can always be fond under `/proc/1`

There is a file at `/proc/PID/status` that provides a lot of information about a process in one file.

Generally it is recommended to use grep to parse files in the `/proc/PID` directory instead of looking for a specific line as the order can vary.

Below are some notable files in the /proc/PID directory

| cmdline	| command line arguments delimited by `\0` |
| cwd | Symbolic link to current working directory |
| environ | Environment list `NAME=value` pairs, delimited by `\0` |
| exe | Symbolic link to file being executed |
| fd | Directory containing symbolic links to files opened by this process |
| maps | Memory mappings |
| mem | Process virtual memory (must `lseek()` to valid offset before I/O) |
| mounts | Mount points for this process |
| root | Symbolic link to root directory |
| status | Various information (process IDs, credentials, memory usage, signals) |
| task | Contains one subdirectory for each thread in process |

### The /proc/PID/fd directory

Each symbolic link found in the `/proc/PID/fd` directory has a name that matches the descriptor number; for example `/proc/1968/fd/1` is a symbolic link to the standard output of process 1968.

As a convenience a process can access its own /proc/PID directory using the symbolic link `/proc/self`

### Threads: the /proc/PID/task directory
  
Under `/proc/PID/task/TID` (where `TID` stands for Thread ID) is a set of files and directories exactly like those that are found under /proc/PID. Since threads share many attributes with their parent you will have a similar layout, but you will also have distinct information there.

### System Information Under /proc

Various files and subdirectories under /proc provide system wide information.  Some of these are…

* `/proc` - Various system information
* `/proc/net` - Status information about networking and sockets
* `/proc/sys/fs` - Settings related to file systems
* `/proc/sys/kernel` - Various general kernel settings
* `/proc/sys/net` - Networking and sockets settings
* `/proc/sys/vm` - Memory management settings
* `/proc/sysvipc` - Information about System V IPC objects
  
### Accessing /proc File
  
It’s a common practice to access files in `/proc` as part of scripts. Some files under `/proc` will be read only. Some `/proc` files can only be accessed by the file owner or a privileged process. Other than the files owned by the owner of a process most files under `/proc` will be owned by root

## Signals Fundamental Concepts

### Concepts and overview

A signal is a notification to a process that an event has occurred. Signals are sometimes described as **software interrupts**. Signals are analogous to hardware interrupts in that they interrupt the normal flow of execution of a program. In most cases it is not possible to predict exactly when a signal will arrive. One process can (if it has permission) send a signal to another process. In this use case signals can be employed as a synchronization technique, or even as a primitive form of inter process communication. It is also possible for a process to send a signal to itself. The usual source of many signals sent to a process is from the kernel. The following are some events that can cause kernel to send a signal to a process.

* A hardware exception occurred, meaning that the hardware detected a fault condition that was notified to the kernel which in turn sent a corresponding signal to the process concerned. Examples of hardware exceptions are …
  * executing a malformed machine language instruction
  * dividing by 0
  * referencing part of memory that is inaccessible.
  * User typing one of the terminal characters that generate signals.
  * interrupt usually Control-C or suspend usually Control-Z
  * A software event occurred.  For example, input became available on a file descriptor, the terminal window has resized, a timer went off, the process's CPU time limit was exceeded, or a child of this process terminated.

Each signal is defined as a unique (small) integer, starting sequentially form 1. These integers are defined in `signal.h` with symbolic names of the form `SIGxxxx`. Since the actual number vary in implementations the symbolic names are used in programs.

Signals fall into two broad categories. The first set contains the traditional or standard  signals, which are used by the kernel to notify processes of vents. On Linux the standard signals are numbered from 1 to 31. The second set consists of realtime signals.

A signal is said to be generated by some event. Once generated, a signal is later delivered to a process which then takes some action in response to the signal. Between the time a signal is generated and the time it is delivered, a signal is said to be **pending**. Normally a pending signal is delivered to a process as soon as it is next scheduled to run or immediately if the process is already running (if process sent a signal to itself). Sometimes you need to make sure a segment of code is not interrupted by a signal. To do so we can add a signal to the process's signal mask (a set of signals whose delivery is blocked). If a signal is generated while its blocked it remains pending until it is later unblocked (removed from signal mask). Upon delivery of a signal, a process carries out one of the following default actions, depending on the signal.

* The signal is ignored; that is it is discarded by the kernel and has no effect on the process (The process never knows that it occurred). 
* The process is terminated (killed).  This is sometimes referred to as abnormal process termination.  As opposed to normal process exit via exit() call.
* A core dump file is generated, and the process is terminated. A core dump file contains an image of the virtual memory of the process which can be loaded into a debugger in order to inspect the state of the process at the time that it terminated.
* The process is stopped - execution of the process is suspended.
* Execution of the process is resumed after previously being stopped.
* Instead of accepting the default for a particular signal, a program can change the action that occurs when the signal is delivered. This is known as setting the disposition of the signal. A program can set one of the following dispositions for a signal.
  * The default action should occur.  This is useful to undo an earlier change of the disposition of the signal to something other than its default.
  * The signal is ignored - this is useful for a signal whose default action would be to terminate the process.
  * A signal handler is executed. A signal handler is a function, written by the programmer that performs appropriate tasks in response to the delivery of a signal. For example the shell has a handler for the `SIGINT` signal generated by the interrupt character Contorl-C that causes it to stop what it is currently doing and return control to the main input loop. Notifying the kernel that a handler function should be invoked is usually referred to as installing or establishing a signal handler. When a signal handler is invoked in response to the delivery of a signal, we say that the signal has been handled or caught.

Signal delivery is typically asynchronous, meaning that the point at which the signal interrupts execution of the process is unpredictable.

### Signal types and default actions

| Signal | Description |
| ------ | ----------- |
| SIGABRT | A process is sent this signal when it calls the abort() function. By default this signal terminates the process with a core dump. This is used to exit a process with a core dump for debugging. |
| SIGALRM | The kernel generates this signal upon the expiration of a real-time timer set by a call to alarm() or settimer(). A real timer is one that counts according to wall clock time. |
| SIGBUS | This signal is generated to indicate certain kinds of memory access errors. For example if you try to access memory outside the memory mapped file allocated to your process. |
| SIGCHLD | This signal is sent by the kernel to a parent process when on of its children terminates. It may also be sent to a process when one of its children is stopped or resumed by a signal. |
| SIGCONT | When sent to a stopped process this signal causes the process to resume. When received by a process that is not currently stopped, this signal is ignored by default. A process may catch this signal so that it carries out some action when resumed. |
| SIGFPE | This signal is generated for certain types of arithmetic error such as divide by 0. FPE stands for floating point error. |
| SIGHUP | When a terminal disconnect (hangup) occurs, this signal is sent to the controlling process of the terminal. A second use of SIGHUP is with daemons. Many daemons are designed to respond to the receipt of a SIGHUP by re-initializing themselves and rereading their configuration files. |
| SIGINT | When the user types the terminal interrupt character (usually Control-C) the terminal driver sends this signal to the foreground process group. The default action for this signal is to terminate the process. |
| SIGKILL | This is the sure kill signal. It can't be blocked, ignored, or caught by a handler, and thus always terminates a process. |
| SIGPIPE | This signal is generated when a process tries to write to a pipe, a FIFO or a socket for which there is no corresponding reader process.  This normally occurs because the reading process has closed its file descriptor for the IPC channel. |
| SIGQUIT | When user types the quit character (usually Control-\) on the keyboard this signal is sent to the foreground process group. By default this terminates the process and causes it to produce a core dump.  Using SIGQUIT in this manner is useful for debugging a program stuck in an infinite loop. |
SIGSEGV | This very popular signal is generated when a program makes an invalid memory reference. A memory reference may be invalid because it references a page that doesn't exist. |
| SIGSTOP | This is the sure stop signal. It can't be blocked, ignored, or caught by a handler; thus it always stops a process. |
| SIGTERM | This is the standard signal used for terminating a process and is the default signal sent by the kill and killall commands. A well designed application will have a handler for SIGTERM that causes the application to exist gracefully. Killing a process with SIGKILL bypasses the SIGTERM handler. |
| SIGTRAP | This signal is used to implement debugger breakpoints and system call tracing as performed by strace() |
| SIGSTP | This is the job control stop signal, sent to stop the foreground process group when the user types the suspend character (usually Control-Z) on the keyboard.  The name of this signal is derived form erminal Stop |
| SIGURG | This signal is sent to a process to indicate the presence of out of band (also known as urgent) data on a socket. |
| SIGXCPU | This signal is sent to a process when it exceeds its CPU time resource limit (RLIMIT_CPU) |

### Introduction to signal handlers

A signal handler also knows as a signal catcher is a function that is called when a specified signal is delivered to a process. Invocation of a signal handler may interrupt the main program flow at any time; the kernel calls the handler on the process's behalf, and when the handler returns, execution of the program resumes at the same point where the handler interrupted it. When the kernel invokes a signal handler, it passes the number of the signal that caused the invocation as an integer argument to the handler. We can establish the same handler to catch different types of signals and use this argument to determine which signal caused the handler to be invoked.

### Sending signals: kill()

One process can send a signal to another process using the `kill()` system call, which is the analog of the kill shell command. The term kill was chosen because the default action for most signals in early Unix systems was to kill the process. The PID argument identifies one or more processes to which the signal is to be sent. Four different cases determine how PID is interpreted.

* If PID is greeter than 0, the signal is sent to the process with the process ID specified by PID
* If PID equals 0, the signal is sent to every process in the same process group as the calling process, including the calling process itself.  
* If PID is less than -1, the signal is sent to all of the processes in the process group whose ID equals the absolute value of the PID.  Sending a signal to all of the processes in a group finds particular use in shell job control.
* If PID equals -1, the signal is sent to every process for which the calling process has permission to send a signal, except init (process ID 1) and the calling process. If a privileged process makes this call, then all processes on the system will be signaled, except for these last two. For obvious reasons, signals sent in this way are sometime called broadcast signals.

The init process (process ID 1), which runs with user and group of root, is a special case.  It can be sent only signals for which it has a handler installed. This prevents the system admin from accidentally killing init, which is fundamental to the operation of the system.

An unprivileged process can send a signal to another process if the real or effective user ID of the sending process matches the real user ID or saved set user-ID of the receiving process. This rule allows users to send signals to set-user-ID programs that they have started, regardless of the current setting of the target process's effective user ID.

The `SIGCONT` signal is treated specially. An unprivileged process may send this signal to any other process in the same session, regardless of user ID checks. This rule allows job-control shells to restart stopped jobs (process groups), even if the processes of the job have changed their user IDs. (i.e. they are privileged processes that have used system calls).

### Checking or existence of a process

The `kill()` system call can serve another purpose. If the sig argument is specified as 0 (the so-called null signal), then no signal is sent. Instead `kill()` merely performs error checking to see if the process can be signaled.  This means that we can use the null signal to see if a process with a specific ID exists. If sending a null signal fails with the error `ESRCH`, then we know the process doesn't exist. If the call fails with the error `EPERM` (meaning the process exists, but we don't have permission to it) or succeeds (meaning we do have permission to it) then we know the process exists.

Other techniques can be used to check whether a particular process is running, including the following.

* Semaphores and exclusive file locks: if the process that is being monitored continuously holds a semaphore or a file lock, then, if we can acquire the semaphore or lock, we know the process has terminated.
* The /proc/PID interface. If the process with the process ID exists than /proc/ID will exist and we can check for that with a `stat()` call.

### The signal mask (blocking signal delivery)
  
For each  process, the kernel maintains a signal mask:k a set of signals whose delivery to the process is currently blocked. If a signal that is blocked is sent to a process, delivery of that signal is delayed until it is unblocked by being removed rom the process signal mask.

A signal may be added to the signal mask in the following ways.

* When a signal handler is invoked, the signal that caused its invocation can be automatically added to the signal mask. Whether or not this occurs depends on the flags used when the handler is established using `sigaction()`
* When a signal handler is established with `sigaction()` it is possible to specify an additional set of signals that are to be blocked when the handler is invoked. The `sigprocmask()` system call can be used at any time to explicitly add signals to, and remove signals from, the signal mask. If we unblock a pending signal it is delivered to the process immediately.

Attempts to block SIGKILL and SIGSTOP are silently ignored. If we attempt to block these signals, `sigprocmask()` neither honors the request nor generates an error.

### Pending Signals
  
If a process receives a signal that it is currently blocking, that signal is added to the process's set of pending signals.  When and if the signal is later unblocked, it is then delivered to the process.
  
### Signals are not queued
  
The set of pending signals is only a mask. It indicates whether or not a signal has occurred, but not how many times it has occurred. In other words, if the same signal is generated multiple times while it is blocked, then it is recorded in the set of pending signals, and later delivered, just once.

A blocked signal is delivered only once, no matter how may times it is generated. Even if a process doesn't block signals, it may receive fewer signals than are sent to it. This can happen if the signals are sent so fast they arrive before the receiving process has a chance to be scheduled for execution by the kernel, with the result that the multiple signals are recorded just once in the process's pending signal set.
  
### Waiting for a signal: pause()
  
Calling `pause()` suspends execution of the process until the call is interrupted by a signal handler (or until an unhandled signal terminates the process)

## Signal handlers

### Designing Signal Handlers
  
The signal handler sets a global flag and exits. The main program periodically checks this flag, and if it is set, takes appropriate action. The signal handler performs some type of cleanup and then either terminates the process or uses a non-local goto to unwind the stack and return control to a predetermined location in the main program.
  
### Signals Are Not Queued (Revisited)

If the signal is generated more than once while the handler is executing, then it is still marked as pending, and it will later be delivered only once. We can't reliably count the number of times a signal is generated. Furthermore, we may need to code our signal handlers to deal with the possibility that multiple events of the type corresponding to the signal have occurred.
  
### Reentrant and Async-Signal Safe Functions
  
#### Reentrant and non-reentrant functions

The main program and the signal handler in effect form two independent (although not concurrent) threads of execution within the same process. A function is said to be reentrant if it can safely be simultaneously executed by multiple threads of execution in the same process. In this context "safe" means that the function achieved its expected result, regardless of the state of execution of any other thread of execution. A function may be non-reentrant if it updates global o static data structures.

#### Standard async-signal safe functions

An async-signal-safe function is one that the implementation guarantees to be safe when called from a signal handler. A function is async-signal-safe either because it is reentrant or because it is not interruptible by a signal handler.  

When writing signal handlers we have two choices

* Ensure that the code of the signal handler itself is reentrant and that it calls only async-signal-safe functions.
* Block delivery of signals while executing code in the main program that calls unsafe functions or works with global data structures also updated by the signal handler.

We must not call unsafe functions from within a signal handler.

### Other Methods of Terminating a Signal

* Use `_exit()` to terminate the process. Beforehand, the handler may carry out some cleanup actions. Note that we can't use `exit()` to terminate a signal handler, because it is not one of the safe functions. It is unsafe because it flushes stdio  buffers prior to calling `_exit()`. 
* Use `kill()` or `raise()` to send a signal that kills the process (i.e. signal whose default action is process termination).
* Perform a non-local GoTo from the signal handler.
* Use the abort() function to terminate the process with a core dump.

## Signals Advanced Features

### Core dump files

One way of causing a program to produce a core dump is to type the quit character (usually Control - |), which causes the `SIGQUIT` signal to be generated:

```bash  
ulimit -c unlimited # see below why this is needed.
$ sleep 30
Type Control - |
Quit (core dumped)
ls -l core
-rw-------- 1 mtc users 57344 Nov 30 12:39 core
```

The core dump file gets created in the working directory of the process with the name core. The `/prox/sys/kernel/core_pattern` file controls the naming of all core dump files produced on the system. By default this file contains the string core.  A privileged user can define this file to use format specifiers.

| Format Specifier | Replaced By |
| ---------------- | ----------- |
| %c | Core file size soft resource limit (bytes; since Linux 2.6.24) |
| %e | Executable filename without prefix |
| %g | Read group ID of dumped process |
| %h | Name of host system |
| %p | Process ID of dumped process |
| %s | Number of signal that terminated process |
| %t | Time of dump, in seconds since epoch |
| %u | Read user ID of dumped process |
| %% | A single % character |

Circumstances in which core dump files are not produced

* The process doesn't have permission to write the core dump file. No write permission to directory or core file exists already and no permission to over write.
* A regular file with the same name already exists.
* The directory in which the core dump file is to be created doesn't exist.
* The process resource limit on the size of a core dump file is set to 0.
* In example above we use ulimit command to ensure that there is no limit on the size of core files.
* The binary executable file that the process is executing doesn't have read permission enabled. This prevents users from using a core dump to obtain a copy of the code of a program that they would otherwise be unable to read.
* The file system on which the current working directory resides is mounted read-only, if full, or has run out of i-nodes.  Alternatively the user has reached their quota limit on the file system.
* Set-user-ID programs executed by a user other than the file owner (group owner) don't generate core dumps. This prevents malicious users from dumping memory of a secure program and examining it for sensitive information such as passwords.
* The `/proc/PID/coredump_filter` can be used on a per process basis to determine which types of memory mappings are written to a core dump file. The value in this file is a mask of four bits corresponding to the four types of memory mappings outlined below. The default value of the file provides traditional Linux behavior: only private anonymous and shared anonymous mappings are dumped.
  * private anonymous mappings
  * private file mappings
  * shared anonymous mappings
  * shared file mappings

### Special Cases for Delivery, Disposition, and Handling
  
#### SIGKILL and SIGSTOP

It is not possible to change the default action for SIGKILL, which always terminates a process, and SIGSTOP which always stops a process. These two signals also can't be blocked.  This is a deliberate design decision.  Disallowing changes to the default actions of these signals means that they can always be used to kill or stop a runaway process.

#### SIGCONT and stop signals

If a process is currently stopped, the arrival of a SIGCONT signal always causes the process to resume, even if the process is currently blocking or ignoring SIGCONT. This feature is necessary because it would otherwise be impossible to resume such stopped processes.  (if the stopped process was blocking SIGCONT, and had established a handler for SIGCONT, then after the process is resumed, the handler is invoked only when SIGCONT is later unblocked)

If any other signal is sent to a stopped process, the signal is not actually delivered to the process until it is resumed via receipt of a SIGCONT signal. The one exception is SIGKILL, which always kills a process even one that is currently stopped.

Whenever SIGCONT is delivered to a process, any pending stop signals for the process are discarded (i.e, the process never sees them). Conversely if any of the stop signals is delivered to a process, then any pending SIGCONT signal is automatically discarded. These steps are taken in order to prevent the action of SIGCONT signal from being subsequently undone by a stop signal that was actually sent beforehand, and vice versa.

### Interruptible and Uninterruptible Process Sleep States

At various times the kernel may put a process to sleep, and the two sleeps states are distinguished:

#### TASK_INTERUPTIBLE

The process is waiting for some event. For example, it is waiting for terminal input, for data to be written to a currently empty pipe, or for the value of a System V semaphore to be increased. A process may spend an arbitrary length of time in this state. If a signal is generated for a process in this state, then the operation is interrupted and the process is woken up by the delivery of a signal. When listed by ps(1) process in the `TASK_INTERRUPTIBLE` state are marked by the letter S in the STAT (process state) field.

#### TASK_UNINTERRUPTIBLE

The process is waiting on certain special classes of event such as the completion of a disk I/O. If a signal is generated for a process in this state, then the signal is not delivered until the process emerges from this state. Processes in the `TASK_UNINTERRUPTIBLE` state are listed by ps(1) with a D in the STATE field.

Because a process normally spends only very brief periods in the `TASK_UNINTERRUPTIBLE` state, the fact that a signal is delivered only when the process leaves this state is invisible. However in rare circumstances a process may remain hung in this state. This can happen as a result of a hardware failure, an NFS problem, or a kernel bug. In such cases, SIGKILL won't terminate the hung process. If the underlying problem can't otherwise be resolved, then we must restart the system in order to eliminate the process.

Starting with kernel 2.6.25, Linux adds a third state `TASK_KILLABLE` This state is like `TASK_UNINTERRUPTILBE`, but wakes the process if a fatal signal (i.e, one that would kill the process is received) By converting relevant parts of the kernel code to use this state, various scenarios where a hung process requires a system restart can be avoided.

### Hardware generated signals

`SIGBUS`, `SIGFPE`, `SIGILL`, and `SIGSEGV` can be generated as a consequence of a hardware exception or less usually by being sent by kill. In the case of a hardware exception, SUSv3 specified that the behavior of a process is undefined if it returns from a handler for the signal, or if it ignores or blocks the signal. The reason for this is as follows. The correct way to deal with hardware generated signals is to either accept their default action (process termination) or to write handlers that don't perform a normal return.  The reasons for this are detailed below.

#### Returning from the signal handler.

Suppose that a machine-language instruction generates one of these signals, and a signal handler is consequently invoked. On normal return from the handler, the program attempts to resume execution at the point where it was interrupted. But this is the very instruction that generated the signal in the first place, so the signal is generated once more. The consequence is usually that the program goes into an infinite loop, repeatedly calling the signal handler.

#### Ignoring the signal

It makes little sense to ignore a hardware-generated signal, as it is unclear how a program should continue execution after, say an arithmetic exception. When one of these signals is generated as a consequence of a hardware exception, Linux forces its delivery, even if the program has requested that the signal be ignored.

#### Blocking the signal

As with the previous case, it makes little sense to block a hardware-generated signal, as it is unclear how a program should then continue execution. On Linux 2.4 and earlier, the kernel simply ignores attempts to block a hardware-generated signal; the signal is delivered to the process anyway, and then either terminates the process or is caught by a signal handler, if one has been established. Starting with Linux 2.6, if the signal is blocked, then the process is always immediately killed by that signal, even I the process has installed a handler for the signal.

### Timing and Order of Signal Delivery

#### When is a signal delivered?

When a process sends itself a signal using raise(), the signal is delivered before the raise() call returns. When a signal is generated asynchronously, there may be a (small) delay while the signal is pending between the time when it was generated and the time it is actually delivered, even if we have not blocked the signal. The reason for this is that the kernel delivers a pending signal to a process only at the next switch from kernel mode to user mode while executing that process. In practice this means the signal is delivered at one of the following times. When the process is rescheduled after it earlier timed out (i.e at the start of a time slice) or at completion of a system call (delivery of the signal may cause blocking system call to complete prematurely)

#### Order of delivery of multiple unblocked signals

As currently implemented, the Linux kernel delivers the signals in ascending order. For example, if pending `SIGINT` (signal 2) and `SIGQUIT` (signal 3) were both simultaneously unblocked then SIGINT would be delivered before `SIGQUIT`, regardless of the order in which the two signals were generated. We can't rely on (standard) signals being delivered in any particular order, since SUSv3 says that delivery order of multiple signals is implementation defined. When multiple unblocked signals are awaiting delivery, if a switch between kernel mode and user mode occurs during the execution of a signal handler, then the execution of that handler will be interrupted by the invocation of a second signal handler (and so on).

#### Realtime signals

Realtime signals were defined in POSIX.1b to remedy a number of limitation of standard signals. They have the following advantages over standard signals. Realtime signals provide an increased range of signals that can be used for application-defined purposes. Only two standard signals are freely available for application-defined purposes: `SIGUSR1` and `SIGUSER2`

Realtime signals are queued. If multiple instances of a realtime signal are sent to a process, then the signal is delivered multiple times. By contrast if we send further instances of a standard signal that is already pending for a process, that signal is delivered only once.

When sending a realtime signal, it is possible to specify data (an integer or pointer value) that accompanies the signal.  The signal handler in the receiving process can retrieve this data. The order of delivery of different realtime signals is guaranteed.  If multiple different realtime signals are pending, then the lowest numbered signal is delivered first. In other words, signals are prioritized with lower numbered signals having higher priority. When multiple signals of the same type are queued, they are delivered along with their accompanying data in the order which they were sent.
  
##### Limits on the number of queued signals

Queuing realtime signals (with associated data) requires that the kernel maintain data structures listing the signals queued to each process. Since the data structures consume kernel memory, the kernel places limits on the number of realtime signals that may be queued.

### Inter process communication with signals

From one viewpoint we can consider signals as a form of inter process communication (IPC) however signals suffer a number of limitations as an IPC mechanism. Programming with signals is cumbersome and difficult for the following reasons. There are better IPC methods that we will cover later. The asynchronous nature of signals means that we face various problems, including re-entrancy requirements, race condition and the correct handling of global variables from signal handlers. Standard signals are not queued. Even for realtime signals, there are upper limits on the number of signals that may be queued. This means that in order to avoid loss of information, the process receiving the signals must have a method of informing the sender that it is ready to receive another signal. The most obvious method of doing this is for the receiver to send a signal to the sender. A further problem is that signals carry limited information.

## Process Creation

### Overview of fork(), exit(), wait(), and execve()

The `fork()` system call allows one process, the parent, to create a new process, the child. This is done by making the new child process an (almost) exact duplicate of the parent. The child obtains copies of the parent's stack, data, heap and text segments. The term fork derives from the fact that we can envisage the parent process as dividing to yield two copies of itself.

The `exit(status)` library function terminates a process, making all resources (memory, open file descriptors, and so on) used by the process available for subsequent reallocation by the kernel. The status argument is an integer that determines the termination status of the process. Using the `wait()` system call, the parent can retrieve this status.

The `wait(&stats)` system call has two purposes. First, if a child of this process has not yet terminated by calling `exit()`, then `wait()` suspends execution of the process until one of its children has terminated.Second the termination status of the child is returned in the status arguments of `wait()`.

The `execve(pathname, argv, envp)` system call loads a new program (pathname, with argument list argv, and environment list envp) into a process's memory. The existing program text is discarded, and the stack, data, and heap segments are freshly created for the new program. This operation is often referred to as `execing` a new program.

Putting it together thing of a shell executing a command.  The shell continuously executes a loop that reads a command, performs various processing on it, and then forks a child process to exec the command. The use of `execve()` is optional as sometimes its useful to have the child carry on executing the same program as the parent.  In either cae the execution of the child is ultimately terminated by a call to `exit()` (or by delivery of a signal), yielding a termination status that the parent can obtain via `wait()`. The call to wait() is likewise optional.  The parent can simply ignore its child and continue executing.  However, we'll see later that the use of wait() is usually desirable, and is often employed within  a handler for the `SIGCHLD` signal, which the kernel generates for a parent process when one of its children terminates.

### Creating a new process: fork()

In many applications creating multiple processes can be a useful way of dividing up a task. For example, a network server process may listen for incoming client requests and create a new child process to handle each request; meanwhile, the server process continues to listen for further client connection. Dividing tasks up in this way often makes applications design simpler.  It also permits greater concurrency (i.e. more tasks or requests can be handled simultaneously). The `fork()` system call creates a new process, the child, which is an almost exact duplicate of the calling process, the parent. The key point to understanding `fork()` is to realize that after it has completed its work, two processes exist, and in each process execution continues from the point where `fork()` returns. The two processes are executing the same program text, but they have separate copies of the stack, data, and heap segments. The child's stack, data, and heap segments are initially exact duplicates of the corresponding parts of the parent's memory. After the `fork()`, each process can modify the variables in its stack, data, and heap segments without affecting the other process. Within the code of a program, we can distinguish the two processes via the value returned from `fork()`. For the parent `fork()` returns the process ID of the newly created child. This is useful because the parent may create and thus track several children via `wait()` or one of its relatives. For the child, `fork()` returns 0. If necessary the child can obtain its own process ID using `getpid()`, and the process ID of its parent using `getppid()`. It is important to realize that after a fork(), it is indeterminate which of the two processes is next scheduled to use the CPU.
  
#### File Sharing Between Parent and Child

When a `fork()` is performed, the child receives duplicates of all of the parent's file descriptors. These duplicates are made in the manner of `dup()`, which means that corresponding descriptors in the parent and the child refer to the same open file description. The open file description contains the current file offset (as modified by `read()`, `write()`, `lseek()`) and the open file status flags (set by `open()` and changed by `fcntl()`). Consequently, these attributes of an open file are shared between the parent and child. For example, if the child updates the file offset, this change is visible though the corresponding descriptor in the parent. Sharing of open file attributes between the parent and child process is frequently useful. For example, if the parent and child are both writing to a file, sharing the file offset ensures that the two process don't overwrite each other's output. It does not, however, prevent the output of the two processes from being randomly intermingled.

#### Memory Semantics of fork()

Making a full copy of the text, data, and heap when executing fork() would be wasteful. Main reason is that fork() is frequently followed by execve() which replaces the process' data heap and stack. Most modern Linux interpretations use two techniques to avoid wasteful copying. The kernel marks the text segment of each process as read-only, so that a process can't modify it sown code. This means that the parent and child can share the same text segment. The fork() system call creates a text segment for the child by building a set of per-process page-table entries that refer to the same physical memory page frames already used by the parent.

For pages in the data, heap and stack segments of the parent process, the kernel employs a technique known as copy on write. Initially the kernel sets things up so that the page-table entries for these segments refer to the same physical memory pages as the corresponding page-table entries in the parent, and the pages themselves are marked read-only. After the fork(), the kernel traps any attempts by either the parent or the child to modify one of these pages, and makes a duplicate copy of the about to be modified page. This new page copy is assigned to the faulting process, and the corresponding page-table entry for the other process is adjusted appropriately. From this point on, the parent and child can each modify their private copies of the page, without the changes being visible to the other process.

### Avoiding Race Conditions by Synchronizing with Signals
  
After a fork(), if either process needs to wait for the other to complete an action, then the active process can send a signal after completing the action; the other process waits for the signal.

## Process Termination

### Terminating a Process: _exit() and exit()

A process can terminate either abnormally (caused by delivery of a signal whose default action is to terminate the process) or via an exit call. A process is always successfully terminated by _exit() (i.e `_exit()` never returns). Performing a return without specifying a value, or falling off the end of the main() function, also results in the caller of `main()` invoking `exit()`, but with results that vary depending on the version of the C standard supported and compilation options employed.
  
### Details of Process Termination

During both normal and abnormal termination of a process, the following actions occur. Open file descriptors, directory steams, message catalog descriptors and conversion descriptors are closed. As a consequence of closing file descriptors, any file locks help by this process are released. Any attached System V shared memory segments are detached, and the `shm_nattch` counter corresponding to each segment is decremented by one. For each system V semaphore for which a `semadj` value has been set by the process, that `semadj` value is added to the semaphore value. If this is the controlling process for a controlling terminal, then the SIGHUP signal is sent to each process in the controlling terminal's foreground process group, and the terminal is disassociated from the session. Any POSIX named semaphores that are open in the calling process are closed as though sem_close() were called. Any POSIX message queues that are open in the calling porcess are closed as though mq_close() were called. If, as a consequence of this process exiting, a process group becomes orphaned and there are any stopped processes in that group, then all process in the group are sent a SIGHUP signal followed by a SIGCONT signal. Any memory locks established by this process using `mlock()` or  `mlockall()` are removed. Any memory mappings established by this process using `mmap()`  are unmapped.

### Exit Handlers
  
An exit handler is a programmer-supplied function that is registered at some point during the life of the process and is then automatically called during normal process termination via `exit()`. Exit handlers are not called if a program calls `_exit()` directly or if the process is terminated abnormally by a signal.

## Monitoring child processes

### Waiting on a child process

#### The wait() System Call

The `wait()` system call waits for one of the children of the calling process to terminate and return the termination status of that child in the buffer pointed to by status.

`pid_t wait(int *status);`

The `wait()` system call does the following

* If no (previously un-waited for) child of the calling process has yet terminated, the call blocks until one of the children terminates.
* If status is not NULL, information about how the child terminated is returned in the integer to which status points.
* The kernel adds the process CPU times and resource usage statistics to running totals for all children of this parent process.
* As its function result, wait() returns the process ID of the child that has terminated.
* On error, wait() returns -1.  One possible error is that the calling process has no (previously un-waited-for) children, which is indicated by the errno value ECHILD.

We can use the following loop to wait for all children of the calling process to terminate.

```c
while((childId = wait(NULL)) != -1)
  continue;
if(errno != ECHILD) /* An unexpected error… */
  errExit("wait");
```

#### The waitpid() System Call

The `wait()` system call has a number of limitations, which `waitpid()` was designed to address. If a parent process has created multiple children, it is not possible to wait() for the completions of a specific child; we can only wait for the next child that terminates. If no child has yet terminated, wait() always blocks. Sometimes, it would be preferable to perform a non-blocking wait so that if no child has yet terminated, we obtain an immediate indication of this fact. Using `wait()`, we can find out only about children that have terminated.  It is not possible to be notified when a child is stopped by a signal (such as `SIGSTOP` or `SIGINT`) or when a stopped child is resumed by delivery of a `SIGCONT` signal. The return value and status arguments of `waitpid()` are the same as for wait().  The pid `arugment` enables the selection of the child to be waited for, as follows.

* If pid is greater than 0, wait for the child whose process ID equals pid.
* If pid equals 0, wait for any child in the same process group as the caller (parent)
* If pid is less than -1, wait for any child whose process group identifier equals the absolute value of pid
* If pid equals -1, wait for any child.  The call `wait(&status)` is equivalent to the call `(waitpid(-1,&status0))`

#### The Wait Status Value

The status value returned by `wait()` and `waitpid()` allows us to distinguish the following events for the child.

* The child terminated by calling _exit() or exit(), specifying an integer exit status.
* The child was terminated by the delivery of an unhandled signal.
* The child was stopped by a signal, and waitpid() was called with the `WUNTRACED` flag
* The child was resumed by SIGCONT signal, and waitpid() was called with `CONTINUED` flag.

## Monitoring child processes (again?)

### Orphans and Zombies
  
The lifetimes of parent and child processes are usually not the same; either the parent outlives the child or vice versa. This raises two questions.
  
* Who becomes the parent of an orphaned child?
  * The orphaned child is adopted by init the ancestor of all processes, whose process ID is 1. This can be used to determine if a child's true parent is till alive (assuming the child was created by a process other than init)

* What happens to a child that terminates before its parent has had a chance to perform a wait()?
  * The point here is that, although the child has finished its work, the parent should still be permitted to perform a wait() at some later time to determine how the child terminated.  
  * The kernel handles this situation by turning the child into a zombie.  
    * This means that most of the resources held by the child are released back to the system to be reused by other processes.
    * The only part of the process that remains is an entry in the kernel's process table recording (among other things) the child process ID, termination status, and resource usage statistics.
  * A zombie process can't be killed by a signal, not even SIGKILL. This ensures that the parent can eventually perform a wait().
  * When the parent does perform a wait(), the kernel removed the zombie, since the last remaining information about the child is no longer required.  
  * On the other hand if the parent terminates without doing a wait(), then the init process adopts the child and automatically performs a wait(), thus removing the zombie process from the system.
  * If a parent creates a child, but fails to perform a wait() then an entry for the zombie child will be maintained indefinitely in the kernel's process table. If a large number of such zombie children are created, they will eventually fill the kernel process table, preventing the creation of new processes.
  * Since zombies can't be killed by a signal, the only way to remove them from the system is to kill their parent (or wait for it to exit) at which time the zombies are adopted and waited on by init, and consequently removed from the system.
  * These semantics have important implications for the design of long-lived parent processes, such as netowrk servers and shells, that create numerous children. Such applications, a parent process should perform wait() calls in order to ensure that dead children are always removed from the system, rather than becoming long-lived zombies. The parent may perform such wait() calls either synchronously or asynchronously in response to delivery of the `SIGCHLD` signal.
  * If you see `<defunct>` in ps output that means the process is a zombie
  
### The SIGCHLD Signal
  
The termination of a child process is an event that occurs asynchronously. A parent can't predict when one of its children will terminate. The parent can call `wait()` or `waitpid()`  without specifying the `WNOHAND` flag, in which case the call will block if a child has not already terminated. The parent can periodically perform a non-blocking check (a poll) for dead children via a call to `waitpid()` specifying the `WNOHANG` flag.

Both of these approaches can be inconvenient. On the one hand, we may not want the parent to be blocked waiting for a child to terminate. On the other hand , making repeated non-blocking waitpid() calls wastes CPU time and adds complexity to an application design. To get around these problems, we can employ a handler for the `SIGCHLD` signal
  
#### Establishing a Handler for SIGCHLD

The `SIGCHLD` signal is sent to a parent process whenever one of its children terminates. By default, this signal is ignored, but we can catch it by installing a signal handler. Within the signal handler, we can use `wait()` (or similar) to reap the zombie child.

There are some subtleties to consider with this approach

* When a signal handler is called, the signal that caused its invocation is temporality blocked (unless the sigaction() SA_NODEFER flag was specified)
* The standard signals, of which SIGCHLD is one, are not queued.
* Consequently if a second and third child terminate in quick succession while a SIGCHLD handler is executing for an already terminated child, then although SIGCHLD is generated twice, it is queued only once to the parent. As a result, if the parent's SIGCHLD handler called wait() only once each time it was invoked, the handler might fail to reap some zombie children. The solution is to loop inside the SIGCHLD handler, repeatedly calling waitpid() with the WNOHANG flag until there are no more dead children processes to be reaped.

Often the body of a SIGCHLD handler simply consists of the following code, which reaps any dead children without checking their status:

```c
while(waitpid(-1, NULL, WNOHANG) > 0)
  continue;
```

The above loop continues until `waitpid()` returns either 0 indicating no more zombie children, or -1 indicating an error (probably ECHILD, meaning that there are no more children).

#### Ignoring Dead Child Processes

Explicitly setting the disposition of SIGCHLD to SIG_IGN causes any child process that subsequently terminates to be immediately removed from the system instaed of being converted into a zombie. In this case, since the status of the child process is simply discarded, a subsequent call to wait() (or similar) can't return any information for the terminated child.

## Program Execution
