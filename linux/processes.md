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

