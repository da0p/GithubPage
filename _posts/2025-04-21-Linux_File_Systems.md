---
title: "Linux Series: File Systems"
date: 2025-04-21
---

A file system is an organized collection of regular files and directories.

## Disks and Partitions

Regular files and directories typically reside on hard disk devices

### Disk Drives

A hard disk drive is a mechanical device consisting of one or more platters that rotate at high speed

Information on the disk surface is located on a set of concentric circles called _tracks_. Tracks
themselves are divided into a number of _sectors_, each of which consists of a series of _physical_
blocks.

### Disk Partitions

Each disk is divided into one or more (nonoverlapping) _partitions_. Each partition is treated by
the kernel as a separate device residing under the _/dev_ directory.

A disk partition may hold any type of information, but usually contains one of the followings:

- a _file system_ holding regular files and directories
- a _data area_ accessed as a raw-mode device
- a _swap area_ used by the kernel for memory management

## File Systems

The basic unit for allocating space in a file system is a _logical_ block, which is some multiple of
contiguous physical blocks on the disk device on which the file system resides

![File System Layout](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/filesystem_structure.drawio.png)

A file system contains the following parts:

- _Boot block:_ This is always the first block in a file system. The boot block is not used by the
file system, it contains information used to boot the operating system. Although only one boot block
is needed by the operating system, all file systems have a boot block (most of which are unused)
- _Superblock:_ This is a single block, immediately following the boot block, which contains
parameter information about the file system, including size of the i-node table, size of logical
blocks in this file system, and size of the file system in logical blocks.
- _I-node table:_ Each file or directory in the file system has a unique entry in the i-node table.
This entry records various information about the file
- _Data blocks:_ the great majority of space in a file system is used for the blocks of data that
form the files and directories residing in the file system

## I-nodes

A file system's i-node table contains one i-node (short for _index node_) for each file residing in
the file system. I-nodes are identified numerically by their sequential location in the i-node table.
The information maintained in an i-node includes the following:

- File type (e.g., regular file, directory, symbolic link, character device)
- Owner (also referred to as the user ID or UID) for the file
- Group (also referred to as the group ID or GID) for the file
- Access permissions for three categories of user: _owner_, _group_, and _other_
- Three timestamps: time of last access to the file, time of last modification of the file, and time
of last status change
- Number of hard links to the file
- Size of the file in bytes
- Number of blocks actually allocated to the file
- Pointers to the data blocks of the file

### ext2

![ext2](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/etx2.drawio.png)

## Virtual File System (VFS)

_Virtual file system_ is a kernel feature that creates an abstraction layer for file-system operations:

- VFS defines a generic interface for file-system operations.
- Each file system provides an implementation for the VFS interface

![VFS](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/vfs.drawio.png)

## Journaling File Systems

Traditional file system suffers from a classic limitation: after a system crash, a file-system
consistency check must be perform on reboot in order to ensure the integrity of the file system.
However, this check requires examining the entire file system, that means it takes a long time to
finish.

Journaling file systems eliminate the need for lengthy file-system consistency checks after a system
crash. A journaling file system logs all metada updates to a special on-disk journal file before they
are actually carried out. The updates are logged in groups of related metadata updates. In the event
of a system crash in the middle of a transaction, on system reboot, the log can be used to rapidly
redo any incomplete updates and bring the system back to a consistent state.

One disadvantage of journaling is that it adds time to file updates

## Virtual Memory File System: tmpfs

_tmpfs_ file system differs from other memory-based file systems in that it is a
_virtual_ memory file system. This means that _tmpfs_ uses not only RAM, but also
the swap space, if RAM is exhausted

If _tmpfs_ file system is unmounted, or the system crashes, then all data in the
file system is lost

_tmpfs_ file systems also serve two special purposes:

- An invisible _tmpfs_ file system, mounted internally by the kernel, is used for
implementing System V shared memory
- A _tmpfs_ file system mounted at _/dev/shm_ is used for the _glibc_ implementation of
POSIC shared memroy and POSIX semaphores