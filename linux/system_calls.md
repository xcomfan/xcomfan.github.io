---
layout: page
title: "System Calls"
permalink: /linux/system_calls
---

A system call is a controlled entry point into the kernel, allowing a process to request that the kernel perform some action on the process's behalf. The `syscalls(2)` manual page lists the Linux system calls. A system call changes the processor state form user mode to kernel mode, so that the CPU can access protected kernel memory. The set of system calls is fixed. Each system call is identified by a unique number. This numbering scheme is not normally visible to programs, which identify system calls by name.

Each system call may have a set of arguments that specify information to be transferred from user space (i.e, the process's virtual address space) to kernel space and vice versa.  From a programming point of view invoking a system call looks much like calling a C function. However, behind the scenes, many steps occur during the execution of a system call. The below steps follow an x86-32 implementation…

1. The application program makes a system call by invoking a wrapper function in the C library

2. The wrapper function must make all of the system call arguments available to the system call trap-handling routine (described below). These arguments are passed to the  wrapper via the stack, but the kernel expects them in specific registers.  The wrapper function copies the arguments to these registers.

3. Since all system calls enter the kernel in the same way, the kernel needs some method of identifying the system call.  To permit this, the wrapper function copies the system call number into a specific CPU register (%eax)

4. The wrapper function executes a trap machine instruction (int 0x80) which causes the processor to switch from user mode to kernel mode and execute code pointed to by location 0x80 (128 in decimal) of the system trap vector.

5. In response to the trap to location 0x80, the kernel invokes its `system_call()` routine (located in the assembler file `/arch/x86/kernel/entry.s`) to handle the trap.  This trap handler…

    a. Saves register values onto the kernel stack

    b. Check the validity of the system call number.

    c. Invokes the appropriate system call service routine, which is found by using the system call number to index to a table of all system call service routines (the kernel variable `sys_call_table`j). If the system call service routine has any arguments, it first checks their validity; for example, it checks that addresses point to valid locations in user memory. Then the service routine performs the required task, which may involve modifying values at addresses specified in the given arguments and transferring data between user memory and kernel memory (e.g, in I/O operations) Finally the service routine returns a result status to the `system_call()` routine.

    d. Restores register values from the kernel stack and places the system call return value on the stack.

    e. Returns to the wrapper function, simultaneously returning the processor to user mode.

6. If the return value of the system call service routine indicated an error, the wrapper function sets the global variable `errno` Using this value wrapper function returns to the caller, providing an integer return value indicating the success or failure of the system call.
