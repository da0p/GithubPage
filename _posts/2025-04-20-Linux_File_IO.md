---
title: "Linux Series: File I/O"
date: 2025-04-20
---

## Universal I/O Model

All system calls for performing I/O refer to open files using a _file descriptor_, a (usually small)
non-negative integer. File descriptors are used to refer to all types of open files, including pipes,
FIFOs, sockets, terminals, devices, and regular files. Each process has its own set of file descriptors

Four system calls: _open()_, _read()_, _write()_, _close()_ are used to perform I/O on all types of files,
including devices such as terminals

Universality of I/O is achieved by ensuring that each file system and device driver implements the same set
of I/O system calls.

With operations outside the universal I/O model, _iotctl()_ system call can be used

## Atomicity and Race Conditions

There are situations that can create race conditions related to files

**Creating File Exclusvely:**
If we check the file existence before creating in two steps. Two processes can have race
condition while creating a file.

![Race condition](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/io_race_conditions.drawio.png)

**Appending Data To A File:**
The same situation can happen if two processes trying to append texts to the end of a file
by using _lseek()_ to set the file offset to the end, then append

In order to solve the problems, atomicity is mandatory. For the above cases, _open()_ provides two flags:

- _O\_EXCL_ in conjunction with _O\_CREAT_, this combination will cause _open()_
to return an error if the file already exists
- _O\_APPEND_ in _write()_ will set file offset to the end and write atomically

## Relationship Between File Descriptors and Open Files

It is possible and useful to have multiple descriptors referring to the same open file. These file
descriptors may be open in the same process or in different processes.

The Linux kernel maintains three data structures:

- the per-process file descriptor table
- the system-wide table of file descriptions
- the file system i-node table

The system-wide table of file descriptions holds the following information:

- the current file offset
- status flags specified when opening the file
- file access mode
- settings relating to signal-driven I/O
- a reference to the _i-node_ object for this file

Besides each file system has a table of _i-node_ for all files residing in the file system
and it contains the following information:

- file type and permissions
- a pointer to a list of locks held on this file
- various properties of the file, including its size and timestamps relating to different types of operations

![File descriptors table and open file table](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/open_file_table.drawio.png)

**Use cases:**

- In process A, _fd 1_ and _fd 20_ point to the same label 23 in the _open file table_, this is the result of _dup()_
- _fd 2_ in process A and _fd 2_ in process B point to the same label 73 in the _open file table_, this can be the result of _fork()_
- _fd 0_ in process A and _fd 3_ in process B have the same _inode ptr_, this is the result of opening the same file

## Scatter-Gather I/O

The _readv()_ and _writev()_ system calls perform scatter-gather I/O. These functions transfer multiple buffers of data in a single
system call instead of accepting a single buffer of data to be read or written

```C
#include <sys/uio.h>

ssize_t readv(int fd, const struct iovec *iov, int iovcnt);

ssize_t writev(int fd, const struct iovec *iov, int iovcnt);
```

with the data structure iovec is defined as below
```C
struct iovec {
    void *iov_base; /* start address of buffer */
    size_t iov_len; /* number of bytes to transfer to/from buffer */
}
```

![Scatter I/O](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/scatter_io.drawio.png)

**Note:**

- These functions complete atomically
- _writev()_ can write partially. A check for the return value is needed to see if all requested bytes were written
- Advantage of using these functions is convenience and speed