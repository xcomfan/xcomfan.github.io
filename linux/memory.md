---
layout: page
title: "Memory Management"
permalink: /linux/memory
---

## Virtual Memory Management

Memory is a limited resource that the kernel needs to manage in an equitable and efficient fashion. Linux uses virtual memory management which has the following benefits.

* Processes are isolated from each other and from the kernel so that one process can't read or modify memory of another.  
* Only part of a process needs to be kept in memory.  Allowing more processes to run in the RAM available.

**Spatial locality** is the tendency of a program to reference memory addresses that are near those that were recently accessed (because of sequential processing of instructions and sometimes sequential processing of data structures).

**Temporal locality** is the tendency of a program to access the same memory addresses in the near future that is accessed in the recent past (because of loops)

`/proc/kallsysm` - provides addresses of virtual memory mapped into processes, but not accessible to program.

A virtual memory scheme splits the memory used by each program into small, fixed size units called pages.  Correspondingly RAM is divided into a series of page frames of the same size. At any one time only some of the pages of a program need to be resident in physical memory page frames. These pages form the sol called resident set. Copies of the unused pages of a program are in the swap area. When a process references a page that is not currently resident in physical memory, a page fault occurs, at which point the kernel suspends execution of the process while the page is loaded from disk into memory. In order to support this organization, the kernel maintains a page table for each process. The page table describes the location of each page in the process's **virtual address space** (the set of all virtual memory pages available to the process). Each entry in the page table either indicates the location of a virtual page in RAM or indicates that it currently resides on disk. Virtual memory management separate the Virtual address space of a process from the physical. This has many advantages.

* Processes are isolated from one another and the kernel, so that one process can't read or modify the memory of another process or the kernel.  

* Where appropriate two or more processes can share memory.  The kernel makes this possible by having page-table entries in different processes refer to the same pages of RAM. Memory sharing occurs in two common circumstances.

  * Multiple processes executing the same program can share a single (read only) copy of the program code.
  * Processes can share the `shmget()` and `mmap()` system calls to explicitly request sharing of memory regions with other processes.  This is done for Inter-process communication.

You can have memory protection schemes such as page table entries controlling if memory pages are readable, writable, executable or some combination of these. Programmers and tools such as compilers and linkers can utilize memory without having to worry about the physical layout of memory. Because only a part of a program needs to reside in memory, the program can load and run much faster.
