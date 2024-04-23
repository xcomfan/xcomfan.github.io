---
layout: page
title: "System and User Space"
permalink: /linux/user_and_system_space
---

## Kernel Mode and User Mode

Modern processors operate in at least two modes. User mode and kernel mode. Hardware instructions allow switching from one mode to the other.  

Areas of virtual memory can be marked as being part of user space or kernel space.  When running in user mode the CPU cannot access kernel space memory. In kernel mode the CPU can access both user space and kernel space. Attempts to access kernel space in user mode will result in a hardware exception.

Certain operations can only be performed when processor is in kernel mode.  Some examples are…

* heart instruction
* accessing the memory management hardware
* initiating device I/O operation

## Process versus kernel view of the system

### Process view

Many things happen transparently for a process. A process doesn't know where it is located in RAM or even if its in RAM or a swap area. It does not know where on disk the files its accessing are, and can't communicate directly with I/O. Process operates in isolation. It can't directly communicate with another process, create a new process or even end its own existence.

### Kernel view

Kernel facilitates running of all processes on the system, and kernel decides which process will obtain access to the CPU next and for how long. Kernel maintains data structures with information about all running processes and updates these structures as processes are created, change state and terminate. The Kernel maintains low level data structures that map the virtual memory of each process into the physical memory of the computer and the swap areas on disk. All communication between processes is done via mechanisms provided by the kernel. In response to requests from processes the kernel creates and terminates processes. The kernel (in particular device drivers) performs all direct communication with input and output devices transferring information to and from user processes as required.
