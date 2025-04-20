---
title: "Linux Series: File I/O Buffering"
date: 2025-04-20
---

In order to optimize speed and efficiency, I/O system calls and the I/O functions
of the standard C library buffer data when operating on disk files. There are various
techniques for influencing and disabling both types of buffering.

## Kernel Buffering of File I/O: The Buffer Cache

When working with disk files, the _read()_ and _write()_ system calls don't directly initiate
disk access. Instead, they simply copy data between a user-space buffer and a buffer in the kernel
_buffer cache_. That means, the system call is not _synchronized_ with the disk operation.

![File I/O Buffering](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/file_io_buffering.drawio.png)

The aim of this design is to allow _read()_ and _write()_ to be fast since they don't need to wait
on a (slow) disk operation

## Controlling Kernel Buffering of File I/O

It is possible to force flusing of kernel buffers for output files. It is necessary sometimes if an
application must ensure that output really has been written to the disk (or at least to the disk's
hardware cache) before continuing.

_Synchronized I/O completion_ means that an I/O operation that has either been successfully
transferred [to the disk] or diagnosed as unsuccessful. There are two definitions for integrity or
_synchronized I/O completion_:

- _Synchronized I/O data integrity completion_: its main objective is to ensure sufficient information
has been transferred to allow later retrieval of that data to proceed. This means not all metadata is
transferred to the disk before it is considered completed 
- _Synchronized I/O file integrity completion_: during a file update, all metadata is transferred to
disk, even if it is not necessary for the operation of a subsequent read of the file data

![File I/O Buffering](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/io_buffering_summary.drawio.png)
