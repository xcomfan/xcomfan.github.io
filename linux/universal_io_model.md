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

## File I/O Buffering

If we are transferring a large amount of data to or from a file, then by buffering data in large blocks, and thus performing fewer sytem calls, we can greatly improve I/O performance.

File systems can be measured by various criteria such as…

* Performance under heavy user load
* Speed of file creation and deletion
* Time required to search for a file in a large directory
* Space required to store small files
* Maintenance of file integrity in the event of a system crash
* Application specific benchmarks are best to determine what you need for a specific use case
* Linux allows an application to bypass the buffer cache when performing disk I/O thus transferring data directly from user space to a file or disk device.  This is called direct I/O or raw I/O.

### Controlling Kernel Buffering of File I/O

It is possible to force flushing of kernel buffers for output files.  Sometimes this is necessary if an application (e.g. a database journaling process) must ensure that output really has been written to the disk (or at least the disk's hardware cache) before continuing.

### Synchronized I/O data integrity and synchronized I/O file integrity

Synchronized I/O completion means "an I/O operation that has either been successfully transferred [to the disk] or diagnosed unsuccessful"

File metadata includes information such as the file owner and group; file permissions; file size; number of (hard) links to the file; timestamps indicating the time of the last file access, last file modification, and last metadata change and file data block pointers

Synchronized I/O data integrity completion is concerned with ensuring that a file data update transfers sufficient information to allow a later retrieval of that data to proceed.

For a read operation this means that the requested file data has been transferred (from the disk) to the process. If there were any pending write operations affecting the requested data, these are transferred to the disk before performing the read. For a write operation this means that the data specified in the write request has been transferred (to the disk) and all file metadata required to retrieve that data has also been transferred.  The key point to note here is that not all modified file metadata attributes need to be transferred to allow the file data to be retrieved. An example of a modified file metadata attribute that would need to be transferred is the file size if the write operation extended the file.  By contrast modified file timestamps would not need to be transferred to disk before a subsequent data retrieval could proceed.

## File Systems

### Device Special Files (Devices)

A device special file corresponds to a device on the system. Within the kernel, each device type has a corresponding device driver which handles all I/O requests for the device. A device driver is a unit of kernel code that implements a set of operations that (normally) correspond to input and output actions on the associated piece of hardware. The API provided by device drivers is fixed and includes operations corresponding to the system calls `open()`, `close()`, `read()`, `write()`, `mmap()` and `ioctl()`

The fact that each device driver provides a consistent interface, hiding the difference in operation of individual devices, allows for universality of I/O.

Some devices are real, such as mice, disks, and tape drives. Others are virtual meaning that there is no corresponding hardware; rather the kernel provides (via device driver) an abstract device with an API that is the same as a real device.

Devices can be divided into two types:

* Character devices handle data on a character by character basis.  Terminals and keyboards are examples of character devices.

* Block devices handle data a block at a time.  The size of a block depends on the type of device, but is typically a multiple of 512 bytes.  Disks are a common example of block devices.

Device files appear with the file system usually under `/dev` directory.

The superuser can create a device file using the `mknod` command and the `mknod()` system call.

In early versions of linux `/dev` contained entries for all possible devices.  This has been fixed with the `udev` program. Udev relies on the `sysfs` file system, which exports information about devices and other kernel objects into user space via a pseudo-file system mounted under `/sys`

#### Device IDs

Each device file has a major ID number and a minor ID number.  The major ID identifies the general class of device, and is used by the kernel to look up the appropriate driver for this type of device. The minor ID uniquely identifies a particular device within a general class. The major and minor  IDs of a device file can be displayed with the `ls -l` command. A device's major and minor IDs are recorded in the i-node for the device file. Each device driver registers its association with a specific major device ID. This association provides the connection between the device special file and the device driver. The name of the device has no relevance when the kernel looks for the device driver.

### Disks and partitions

The command `fdisk -l` lists all partitions on a disk.

The Linux specific `/proc/partitions` file lists the major and minor device numbers, size and name of each disk.

A disk partition may hold any type of information but usually contains one of the following

* A file system holding regular files and directories.
* A data area accesses as a raw-mode device
* A swap area used by kernel for memory management

The Linux specific `/proc/swaps` file can be used to display information about currently enabled swap areas on the system.

### File system types

The file system types currently known by the kernel can be viewed in the Linux specific `/proc/filesystem` file

A file system contains the following parts...

* Boot block - This is always the first block in a file system.  The boot block is not used by the file system; rather it contains information used to boot the operating system.  Although only one boot block is needed by the operating system, all file systems have a boot block (most of which are unused)
* Superblock - This is a single block, immediately following the boot block, which contains parameter information about the file system, including:
  * The size of the i-node table
  * The size of logical blocks in this file system
  * The size of the file system in logical blocks
* I-node-table - Each file or directory in the file system has a unique entry in the i-node table.  This entry records various information about the file.  more details a bout i-nodes below.
* Data blocks - The majority of space in a file system is used for the blocks of data that form the files and directories that reside on the file system.

Different file systems residing on the same physical device can be of different types and sizes, and have different parameter settings (e.g. block size). This is one of the reasons for splitting a disk into multiple partitions.

### I-nodes

* A file systems i-node table contains one i-node (short for index node) for each file residing in the file system.  
* I-nodes are identified numerically by their sequential location in the i-node table.
* The i-node number is the first field displayed by the ls -li command.
* The information contained in an i-node includes…
  * File type (regular file, directory, symbolic link, character device)
  * Owner (also referred to as the user ID or UID) of the file
  * Group (also referred to as the group ID or GID) for the file
  * Access permissions for three categories user owner and group.
  * Three timestamps
    * last access shown by ls -lu
    * last modified shown by ls -l 
    * last status change shown by ls -lc
  * most Linux systems do not record creation time.
  * Number of hard links to the file
  * Size of file in bytes
  * Number of blocks allocated to the file measured in 512 byte blocks.  This number is not necessarily the file size as a file can contain holes and thus require fewer allocated blocks.
  * Pointers to the data blocks of the file.

#### I-nodes and data block pointers in ext2

* Like most UNIX file systems, the ext2 file system does not store data blocks of a file contiguously or even in sequential order. It attempts to keep them close but mostly things don't work out this way.
* Under ext2 each i-node contains 15 pointers.
  * First 12 pointers numbered 0 to 11 point to the location in the file system of the first 12 blocks of the file.
  * The next pointer is a pointer to a block of pointers that give the location of the thirteenth and subsequent data blocks of the file.  
  * The number of pointers in this block depends on the block size of the file system. Each pointer is 4 bytes so there may be from 256 pointers for a 1024 block size to 1024 pointers for a 4096 byte block size.
  * This allows for large files but for larger files the fourteenth pointer (number 13) is a double indirect pointer.  It points to a block of pointers that in turn point to a block of pointers that in turn point to data blocks.
  * Should the need arise the last 15th pointer is a triple-indirect pointer
  * This complex system allows i-node to be a fixed size while at the same time allowing for files of arbitrary size
  * Additionally, it allows the file system to store the blocks of a file noncontiguously while allowing the data to be accessed randomly via lseek() the kernel just needs to calculate which pointers to follow

### The Virtual File System (VFS)

Each of the file systems available on Linux differs in the details of its implementation. Such differences include for example the way in which the blocks of a file are allocated and the manner in which directories are organized. If every program that worked with files needed to understand the specific details of each files system the task of writing programs that work with each of these files systems would be nearly impossible. The Virtual File System (VFS) is a kernel feature that resolves this problem by creating an abstraction layer for a file system. The VFS defines a generic interface for file-system operations.  All program that work with files specify their operations in terms of this generic interface. Each file system provides an implementation for the VFS interface. Under this scheme programs need to understand only the VFS interface and ignore details of individual file-system implementations. The VFS interface includes operations corresponding to all of the usual system calls for working with file systems and directories such as…

* open()
* read()
* write()
* lseek()
* close()
* truncate()
* stat()
* mount()
* unmount()
* mmap()
* mkdir()
* link()
* unlink()
* symlink()
* rename()

Not all files systems support all of the VFS operations (for example VFAT doesn't support the notion of symbolic links).  In such cases the underlying file system passed an error code back to the VFS layer indicating the lack of support and the VFS passes this error up to the applications.

### Journaling File Systems

A file system that is not journaled such as ext2 needs to run a file system consistency check (fsck) after a system crash reboot in order to ensure the integrity of the file system. This is necessary because at the time of the crash a file update may have been only partially completed, and the file system metadata (directory entries, i-node information, and file data block pointers) may be in an inconsistent state and the file system may be further damaged if these inconsistencies are not repaired. A file system consistency check ensures the consistency of the file system metadata. Where possible, repairs are performed; otherwise, information that is not retrievable (possibly including file data) is discarded. The problem is that a consistency check requires examining the entire file system. On a small file system, this may take anything from several seconds to a few minutes. On a large file system this may take several hours which is a problem for systems that need high availability.

Journalling file system eliminate the need for lengthy file system consistency checks after a system crash. A journaling file system logs (journals) all metadata updates to a special on-disk journal file before they are actually carried out. The updates are logged in groups of related metadata updates (transactions). In the event of a system crash in the middle of a transaction, on system reboot, the log can be used to rapidly redo any incomplete updates and bring the file system back to a consistent state. To borrow DB terminology we can say that a journaling file system ensures that metadata transactions are always committed as a complete unit. Even very large journaling file systems can typically be available within seconds after a system crash, making them very attractive for systems with high-availability requirements. The most notable disadvantage of journaling is that it adds time to file updates, though good design can make this overhead low.

Journaling File Systems

* XFS
* ext4
  * supports extents (reservations of contiguous blocks of storage)
* Btrfs (B-tree FS)
  * supports extents
  * writable snapshots
  * checksums on data and metadata
  * online file-system checking
  * online file-system defragmentation 
  * space-efficient packing of small files and space-efficient indexed directories.

### Mounting and Unmounting File Systems

A list of currently mounted file systems can be read from the Linux specific `/proc/mounts` virtual file system.

The mount and unmount commands automatically maintain the file `/etc/mtab` which contains information that is similar to that in `/proc/mount` but has slightly more details. The `/etc/mtab` provides file system specific options given to the mount command. This file may be inaccurate if some applications calling mount()/unmount() fail to update it.

`/etc/fstab` is maintained by the system administrator

`/etc/fstab` `/proc/mounts` `/etc/fstab` all share a similar format. `/dev/sda9 /boot ext3 rw 0 0`

* the name of the mounted device
* The mount point of the device
* The file system type
* Mount flags in above example rw means read write
* A number used to control the operation of file-system backups by dump(8).  This files and next one are only in the fstab file. For /proc/mounts and /etc/mtab it is always set to 0.
* A number used to conrol the order in which fsck(8) checks file systems at boot time.
* Flags you can set when mounting (these are ones I though were relevant there are more…)
  * MS_NOATIME - Don't update the last access time for files
  * MS_NODIRATIME - Don't update the last access time for directories
  * MS_NOEXEC - Don't allow programs to be executed
  * MS_NOSUID - Disable set user ID and set group ID programs
  * MS_RDONLY - Read-only mount; files can't be created or modified.

#### Advanced Mount Features

##### Stacked mount points

Mounting a File System at Multiple Mount Points.  You can mount the same device on two mount points

```bash
$ su
mkdir /testfs
mkdir /demo
mount /dev/sda12/testfs
mount /dev/sd12/demo
```

You can stack multiple mounts on a single mount point. Each new mount hides the directory subtree previously visible at that mount point. When the mount at the top of the stack is unmounted, the previously hidden mount becomes visible once more

```bash
mount /dev/sda12 /testfs
touch /testfs/myfile
mount /dev/sda13/testfs
```

One use of mount stacking is to stack a new mount point on an existing mount point that is busy.  Processes that hold file descriptors open that are chroot jailed or that have current working directories within the old mount point continue to operate under that mount, but processes making new access to the mount point use the new mount.  

Combined with a `MNT_DETACH unmount`, this can provide a smooth migration off a file system without needing to take the system into single user-mode.

When stacking or having multiple devices on the same mount point you can have different options per mount.  The options that support this are…

* MS_NOATIME
* MS_NODEV
* MS_NODIRATIMVE
* MS_NOEXEC
* MS_NOSUID
* MS_RDONLY

For xxample

```bash
mount /dev/sda12 /testfs
mount -o noexec /dev/sda12 /demo
```

##### Bind Mounts

A bind mount (created using the mount() MS_BIND flag) allows a directory or a file to be mounted at some other location in the file-system hierarchy. This results in the directory or file being visible in both locations.  

A bind mount is like a hard link but differs in 2 ways

* A bind mount can cross file system mount points (and even chroot jails)
* It is possible to make a bind mount for a directory

We can create a bind mount from the shell using the bind option to mount.

```bash
pwd
/testfs
mkdir d1
touch d1/x
mkdir d2
mount -bind d1 d2
ls d2
```

##### Recursive Bind Mounts

By default if we create a bind mount for a directory using MS_BIND, then only that directory is mounted at the new location.  If there are any sub-mounts under the source directory they are not replicated under the mount target. A recursive bind mount solves this "issue" `mount --rbind top dir2`

##### A Virtual Memory File System: tmpfs

A virtual file system resides in memory but to applications looks just like any other file system. File operations on a virtual file system are much faster. There are multiple memory based file systems on Linux but tmpfs is one of the more sophisticated ones. tmpfs is a virtual file system which means it not only uses RAM but can also use swap space if space is exhausted

To create a tmpfs file system you use the command `mount -t tmpfs source target` for example `mount -t newtmp /tmp` It is not necessary to make mkfs file system first because the kernel automatically builds a file system as part of the mount call.

This can be used to improve performance of applications that need a lot of ephemeral I/O. By default, a tmpfs file system can grow to half the size of RAM, but the size=nbytes mount option can be used to set a different ceiling for the file system size. You can do this at creation or later with a remount command.

***Note:** If you un-mount a tmpfs system than all data is lost

Besides user applications tmpfs serves two special purposes

* An invisible tmpfs file system, mounted internally by the kernel is used for implementing System V shared memory and shared anonymous memory mappings.
* A tmpfs file system mounted at /dev/shm (/run/shm on some systems) is used for the glibc implementation of POSIX shared memory and POSIX semaphores.

##### Information on File Systems

Many native Unix and Linux file systems support the notion of reserving a certain portion of the blocks of a file system for the superuser, so that if the file system fills up, the superuser can still log into the system and do some work to resolve the problem.
If there are reserved blocks in the file system than the difference in values of the `f_bfree` and `f_bavail` fields in the `statfs` structure tells us how many blocks are reserved.

### File Attributes

#### File size, blocks allocated, and optimal I/O block size

512-byte is the smallest bock size (historically) for any file systems that have been implemented under Unix. More modern file systems use larger block sizes but they are multiples of 2 (1024, 2048, 4096)

#### Timestamps

Since kernel 2.6 you could have nanosecond resolution for atime mtime on files. This is supported by JFS, XFS, ext4 and Btrf but not ext2, ext3 and Reiserfs. You can modify a time stamp with the utime() system call. `tar` and `unzip` commands use this to reset file timestamps when unpacking an archive.

#### File Ownership

##### Ownership of New Files

When a new file is created, its user ID is taken from the effective user ID of the process. The group ID of the new file may be taken from either the effective group ID or the group ID of the parent directory Which of these is used is determined by various factors including type of file system.

Ext2 logic for example; when the file system is mounted you can specify the `-o grpid` or `-o nogrpid` options.

* -o grpid means it will inherit permission from parent directory
* -o nogrpid means it will take it from the effective gid of the process.

Rules determining the group ownership of a newly created file.

| File system mount option | set-group-ID bit enabled on parent directory | Group ownership of new file taken from |
| ------------------------ | -------------------------------------------- | -------------------------------------- |
| -o grpid, -o bsdgroups |(ingored)| parent directory group ID |
| -o nogrpid, -o sysvgroups | no | Process effective group ID |
| -o nogrpid, -o sysvgroups | yes | parent directory group ID |

At time of writing only the following file systems support the grpid and nogrpid flags

* ext2, ext3, ext4
* XFSj

Other file systems follow the nogrpid rules.

#### Changing File Ownership

Un unprivileged process can use `chown()` to change the group ID of a file that it owns to any of the groups of which they are a member. If the owner or group of a file is changed, then the set user ID and set group ID permissions bits are both turned off. This is a security precaution to make sure that you cannot enable the set user ID on an executable file and then make it owned by a privileged user or group thereby gaining that privileged identity when executing the file.

When changing the owner or group of a file, the set group ID permission bit is not turned off if the group-execute permission bit is already off or if we are changing the ownership of a directory. In both these cases the set-group ID bit is being used for a purpose other than the creation of a set-group-ID program, and therefore its undesirable to turn the bit off.  

These other users are …

* If the group-execute permission bit is off, then he set-group-ID permission bit is being used to enable mandatory file locking
* In the case of a directory, the set group ID bit is being used to control the ownership of new files created in the directory

#### Permissions on Directories

##### Read

The contents (i.e, the list of filenames) of the directory may be listed (e.g, by ls) if testing be careful that some Linux distros by default alias ls to ls -F and this requires execute permission on the directory. To use an unadulterated ls you can call /bin/ls

We need to have execute permission on the directory to access the I nodes of the files within.

##### Write

Files may be created in and removed from the directory.  Note that it is not necessary to have the permission on the file itself in order to be able to delete it.

#### Execute

Files within the directory may be accessed. Execute permission on a directory is sometimes called search permission. If you have execute permission on a directory but not read than you can access a file inside but only if you know its name.

When accessing a file execute permission is required on all of the directories listed in the pathname.  For example `/home/mtk/x` would require execute permission on `/`, `/home` and `/home/mtk` as well as read permission on the file `x` itself.
