---
title: "Linux Series: Processes"
date: 2025-04-27
---

A process is an instance of an executing program

## Program

A _program_ is a file containing a range of information that describes how to construct a process
at run time. The information includes:

- _Binary format identification:_ Each program file includes metainformation describing the format
of the executable file. This enables the kernel to interpret the remaining information in the file
- _Machine-language instructions:_ These encode the algorithm of the program 
- _Program entry-point address:_ This identifies the location of the instruction at which execution
of the program should commence
- _Data:_ The program file contains values used to initialize variables and also literal constants
used by the program
- _Symbol and relocation tables:_ These describe the location and names of functions and variables
within the program. These tables are used for a variety of purposes, including debugging and run-time
symbol resolution (dynamic linking)
- _Shared-library and dynamic-linking information:_ The program file includes fields listing the
shared libraries that the program needs to use at run time and the pathname of the dynamic linker
that should  be used to load these libraries
- _Other information:_ The program file contains various other information that describes how to
construct a process

## Memory Layout of a Process

The following memory layout of a process is described in _virtual memory_. Linux employs a technique
known as _virtual memory management_. The aim of this technique is to make efficient use of both the
CPU and RAM (physical memory) by exploiting a property that is typical of most programs: _locality of reference_.
There are two kinds of locality:

- _Spatial locality:_ is the tendency of a program to reference memory addresses that are near those
that were recently accessed (because of sequential processing of instructions, and, sometimes,
sequential processing of data structures) 
- _Temporal locality:_ is the tendency of a program to access the same memory addresses in the near
future that it accessed in the recent past (because of loops)

![Memory Layout of Process](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/memory_layout_of_process.drawio.png)

## Virtual Memory Management

A virtual memory scheme splits the memory used by each program into small, fixed-size units called _pages_. 
Correspondingly, RAM is divided into a series of _page frames_ of the same size. At any one time,
only some of the pages of a program need to be resident in physical memory page frames; these pages
form the so-called _resident set_. Copies of the unused pages of a program are maintained in the
_swap area_-a reserved area of disk space used to supplement the computer's RAM-and loaded into
physical memory only as required. When a process references a page that is not currently resident in
physical memory, a _page fault_ occurs, at which point the kernel suspends execution of the process
while the page is loaded from disk into memory.

![Virtual Memory Management](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/virtual_memory_overview.drawio.png)

### Addvantage of Virtual Memory Management

Virtual memory management separates the virtual address space of a process from the physical address
space of RAM. This provides many advantages:

- Processes are isolated from one another and from the kernel, so that one process can't read or
modify the memory of another process or the kernel. This is accomplished by having the page-table
entries for each process point to distinct sets of physical pages in RAM (or in the swap area).
- Where appropriate, two or more processes can share memory. The kernel makes this possible by
having page-table entries in different processes refer to the same pages of RAM. Memory sharing can
happen in two circumstances:
  - Multiple processes executing the same program can share a single (read-only) copy of the
    program code.
  - Processes can use the _shmget()_ and _nmap()_ system calls to explicitly request sharing of
    memory regions with other processes. This is done for the purpose of interprocess communication.
- The implementation of memory protection schemes is facilitated; that is, page-table entries can be
marked to indicate the contents of the corresponding page are readable, writable, executable, or
some combinations of these protections. Where multiple processes share pages of RAM, it is possible
to specify each process has different protections on the memory; for example, one process might
have read-only access to a page, while another has read-write access
- Programmers, tools, such as compilers, linker, don't need to be concerned with the physical layout
of the program in RAM
- Because only a part of a program needs to reside in memory, the program loads and runs faster.
Furthermore, the memory footprint (i.e, virtual size) of a process can exceed the capacity of RAM.
- Since each process uses less RAM, more processes can simultaneously be held in RAM