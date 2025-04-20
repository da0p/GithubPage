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