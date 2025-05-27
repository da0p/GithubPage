---
title: "Linux Series: Memory Mappings"
date: 2025-05-21
---

## Overview

The _mmap()_ system call creates a new _memory mapping_ in the calling process's virtual address space.
A mapping can be of two types:

- _File mapping:_ A file mapping maps a region of a file directly into the calling process's virtual
  memory. Once a file is mapped, its contents can be accessed by operations on the bytes in the
  corresponding memory region. The pages of the mapping are (automatically) loaded from the file as
  required. This type of mapping is also known as _file\-based mapping_ or _memory\-mapped file_
- _Anonymous mapping:_ An anonymous mapping doesn't have a corresponding file. Instead, the pages of
  the mapping are initialized to 0

The memory in one proces's mapping may be shared with mappings in other processes. This can occur in
two ways:

- When two processes map the same region of a file, they share the same pages of physical memory
- A child process created by _fork()_ inherits copies of its parent's mappings, and these mappings
  refer to the same pages of physical memory as the corresponding mappings in the parent

When two or more processes share the same pages, each process can potentially see the changes to the
page contents made by other processes, depending on whether the mapping is _private_ or _shared_

- _Private mapping (MAP\_PRIVATE):_ Modifications to the contents of the mapping are not visible to
  other processes and, for a file mapping, are not carried through to the underlying file
- _Shared mapping (MAP\_SHARED):_ Modifications to the contents of the mapping are visible to other
  processes that share the same mapping and, for a file mapping, are carried through to the underlying 
  file

| Visibility of modifications | File                                                      | Anonymous                              |
| --------------------------- | --------------------------------------------------------- | -------------------------------------- |
| Private                     | Initializing memory from contents of file                 | Memory allocation                      |
| Shared                      | Memory-mapped I/O; sharing memory between processes (IPC) | Sharing memory between processes (IPC) |

- _Private file mapping:_ The contents of the mapping are initialized froma  file region. Multiple
  processes mapping the same file initially share the same physical pages of memory, but the copy-on-write
  technique is employed, so that changes to the mapping by one process are invisible to other processes.
  The main use of this type of mapping is to initialize a region of memory from the contents of a file
- _Private anonymous mapping:_ Each call to _mmap()_ to create a private anonymous mapping yields a
  new mapping that is distinct from other anonymous mappings created by the same (or a different) process.
  The primary purpose of private anonymous mappings is to allocate new (zero-filled) memory for a process
- _Shared file mapping:_ All processes mapping the same region of a file share the same physical pages
  of memory, which are initialized from a file region. Modifications to the contents of the mapping
  are carried through to the file. Two purposes:
  - It permits _memory\-mapped I/O_. A file is loaded into a region of the process's virtual memory,
    and modifications to that memory are automatically written to the file. 
  - Allow unrelated processes to share a region of memory in order to perform (fast) IPC in a manner
    similar to System V shared memory segments
- _Shared anonymous mapping:_ As with private anonymous mapping, each call to _mmap()_ to create a
  shared anonymous mapping creates a new, distinct mapping that doesn't share pages with other mapping.
  The difference is that the pages of the mapping are not copied-on-write. Shared anonymous mappings
  allow IPC only between related processes

Note that mappings are lost when a process performs an _exec()_, but are inherited by the child of a
_fork()_. The mapping type (_MAP\_PRIVATE_ or _MAP\_SHARED_) is also inherited

## File Mappings

To create a file mapping, two steps must be performed:

1. Obtain a descriptor for the file, typically via a call to _open()_
2. Pass that file descriptor as the _fd_ argument in a call to _mmap()_

### Private File Mappings

The two most common uses of private file mappings are the following:

- To allow multiple processes executing the same program or using the same shared library to share the
  same (read-only) text segment, which is mapped from the corresponding part of the underlying executable
  or library file 
- To map the initialized data segment of an executable or shared library. Such mappings are made private
  so that modifications to the contents of the mapped data segment are not carried through to the underlying
  file

Both of these uses of _mmap()_ are normally invisible to a program, because these mappings are created
by the program loader and dynamic linker

![Memory-Mapped File](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/memory_mapped_file.drawio.png)

### Shared File Mappings

When multiple processes create shared mappings of the same file region, they all share the same physical
pages of memory. In addition, modifications to the contents of the mapping are carried through to the
file. In effect, the file is being treated as the paging store for this region of memory.

Shared file mappings serve two purposes: memory-mapped I/O and IPC

![Shared-File Mapping](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/shared_file_mapping.drawio.png)

### Memory-mapped I/O

Since the contents of the shared file mapping are initialized from the file, and any modifications to
the contents of the mapping are automatically carried through to the file, we can perform file I/O simply
by accessing bytes of memory, relying on the kernel to ensure that the changes to memory are propagated
to the mapped file. This technique is referred to as _memory\-mapped I/O_, and is an alternative to
using _read()_ and _write()_ to access the contents of a file

Memory-mapped I/O has two potential advantages:

- By replacing _read()_ and _write()_ system calls with memory accesses, it can simplify the logic of
  some applications
- It can, in some circumstances, provide better performance than file I/O carried out using the conventional
  I/O system calls
The reasons that memory-mapped I/O can provide performance benefits are as follows:
- A normal _read()_ or _write()_ involves two transfers: one between the file and the kernel buffer
  cache, and the other between the buffer cache and a user-space buffer. Using _mmap()_ eliminates
  second of these transfers. For input, the data is available to the user process as soon as the kernel
  has mapped the corresponding file blocks into memory. For output, the user process merely needs to
  modify the contents of the memory, and can then rely on the kernel memory manager to automatically
  update the underlying file
- In addition to saving a transfer between kernel space and user space, _mmap()_ can also improve
  performance by lowering memory requirements. When using _read()_ or _write()_, the data is maintained
  in two buffers: one in user space and the other in kernel space. When using _mmap()_, a single buffer
  is shared betwen the kernel space and user space. Furthermore, if multiple processes are performing
  I/O on the same file, then, using _mmap()_, they can all share the same kernel buffer, resulting in
  an additional memory saving

### IPC Using a Shared File Mapping

Since all processes with a shared mapping of the same file region share the same physical pages of
memory, the second use of a shared file mapping is as a method of (fast) IPC

## Synchronizing a Mapped Region: _msync()_

The kernel automatically carries modifications of the contents of a _MAP\_SHARED_ mapping through to
the underlying file, but, by default, provides no guarantees about when such synchronization will occur.
The _msync()_ system call gives an application explicit control over when a shared mapping is synchronized
with the mapped file. Synchronizing a mapping with the underlying file is useful in various scenarios.

```c
#include <sys/mman.h>

int msync(void *sddr, size_t length, int flags);
```

## Anonymous Mappings

An _anonymous mapping_ is one that doesn't have a corresponding file

### MAP_ANONYMOUS and /dev/zero

Two different, equivalent methods of creating an anonymous mapping with _mmap()_:

- Specify _MAP\_ANONYMOUS_ in flags and specify _fd_ as -1
- Open the _/dev/zero_ device file and pass the resulting file descriptor to _mmap()_
With both the _MAP\_ANONYMOUS_ and the _/dev/zero_ techniques, the bytes of the resulting mapping
are initialized to 0

### MAP_PRIVATE Anonymous Mappings

_MAP\_PRIVATE_ anonymous mappings are used to allocate blocks of process-private memory initialized to 0

### MAP_SHARED Anonymous Mappings

A _MAP\_SHARED_ anonymous mapping allows related processes (e.g., parent and child) to share a region
of memory without needing a corresponding mapped file

### MAP_NORESERVE and Swap Space Overcommitting

Some applications create large (usually private anonymous) mappings, but use only a part of the mapped
region. If the kernel always allocated (or reserved) enough swap space for the whole of such mappings,
then a lot of swap space would potentially be wasted. Instead, the kernel can reserve swap space for
the pages of a mapping only as they are actually required. This approach is called _lazy swap reservation_,
and has the advantage that the total virtual memory used by applications can exceed the total size of
RAM plus swap space

### Nonlinear Mappings: _remap_file_pages()_

There is a possibility to create large numbers of nonlinear mappings - mappings where the pages of the
file appear in a different order within contiguous memory

![Nonlinear File Mapping](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/nonlinear_file_mapping.drawio.png)