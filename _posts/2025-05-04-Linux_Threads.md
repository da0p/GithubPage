---
title: "Linux Series: Threads"
date: 2025-05-04
---

## Overview

Threads are a mechanism that permits an application to perform multiple tasks concurrently. A single
process can contain multiple threads. All of these threads are independently executing the same
program, and they all share the same global memory, including the initialized data, uninitialized
data, and heap segments

![Threads Overview](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/threads_overview.drawio.png)

| Criteria                      | Processes                                                                            | Threads                                                                                                                     |
| ----------------------------- | ------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| Sharing information           | Difficult to share information between processes                                     | Easy, just a matter of copying data into shared (global or heap) variables                                                  |
| Creation                      | Expensive even with copy-on-write technique since the need to duplicate various info | Faster since copy-on-write duplication of pages of memory is not required, nor duplication of page tables                   |
| Isolation                     | More isolated                                                                        | Thread-safe functions must be ensured                                                                                       |
|                               |                                                                                      | A bug in one thread can damage all of the threads in the process, since they share same address space and other attributes  |
| Usage of virtual memory space | Separate processes can employ the full range of available virtual memory             | Each thread competes for use of the finite virtual address space of the host process                                        |
|                               |                                                                                      | Each thread's stack and thread-specific data (or thread-local storage) consumes a part of the process virtual address space |
| Dealing with signals          | Less complicated                                                                     | Requires careful design (usually avoid the use of signals in multithreaded programs)                                        |
| Running program               | Multithreaded application must be running the same program                           | Multiprocess application can run different programs                                                                         |

Some notes:

- If threads are not joined, they will become _zombie_ when they terminated 
- Threads are peers. Any thread in a process can join with any other thread in the process. It is
  no such relationship with processes. Only parent process can wait for their children
- There is no way to join with any thread and do nonblocking join. Thread must join with another
  specific thread and the need of using condition variables to achive nonblocking join