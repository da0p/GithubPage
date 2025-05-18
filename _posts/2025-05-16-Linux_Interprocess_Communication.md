---
title: "Linux Series: Interprocess Communication"
date: 2025-05-16
---

## Overview

UNIX communication and synchronization facilities can be divided into three broad functional categories:

- _Communication:_ These facilities are concerned with exchanging data between processes
- _Synchronization:_ These facilities are concerned with synchronizing the actions of processes or
  threads
- _Signals:_ Signals can be used as a synchronization technique in certain circumstances

![UNIX IPC Taxonomy](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/unix_ipc_taxonomy.drawio.png)

## Communication Facilities

Various communication facilities allow processes to exchange data with one another. They can be
divided into two categories:

- _Data-transfer facilities:_ The key factor distinsguishing these facilities is the notion of writing
  and reading. In order to communicate, one process writes data to the IPC facility, and another process
  reads the data. These facilities require two data transfers between user memory and kernel memory:
  one transfer from user memory to kernel memory during writing, and another transfer from kernel memory
  to user memory during reading
- _Shared memory:_ Shared memory allows processes to exchange information by placing it in a region
  of memory that is shared between the processes. A process can make data available to other processes
  by placing it in the shared memory region. Because communication doesn't require system calls or
  data transfer between user memory and kernel memory, shared memory can provide very fast communication

| **Data Transfer**                                                                                          | **Shared Memory**                                                                      |
| ---------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Reads are destructive. A read operation consumes data, and that data is not available to any other process | Fast communication, but there is a need to synchronize operations                      |
| Synchronization between the read and writer processes is automatic                                         | Semaphore is the usual synchronization method used with shared memory                  |
|                                                                                                            | Data placed in shared memory is visible to all of the processes that share that memory |

### Data Transfer

- _Byte stream:_ The data exchanged via pipes, FIFOs, and datagram sockets is an undelimited byte stream
- _Message:_ The data exchanged via System V message queues, POSIX message queues, and datagram sockets
  takes the form of delimited messages
- _Pseudoterminals:_ A pseudoterminal is a communication facility intended for use in specialized situations

![UNIX Data Transfer](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/unix_data_transfer.drawio.png)

### Shared Memory

Modern UNIX systems provide three flavors of shared memory:

- System V shared memory
- POSIX shared memory
- Memory mappings

## Synchronization Facilities

UNIX systems provide the following synchronization facilities:

- _Semaphores:_ A semaphore is a kernel-maintained integer whose value is never permitted to fall below 0.
  A process can decrease or increase the value of a semaphore. If an attempt is made to decrease the
  value of the semaphore below 0, then the kernel blocks the operation until the semaphore's value
  increases to a level that permits the operation to be performed
- _File locks:_ File locks are a synchronization method explicitly designed to coordinate the actions
  of multiple processes operating on the same file. File locks come in two flavors: read (shared) locks
  and write (exclusice) locks. Any number of processes can hold a read lock on the same file (or
  region of a file). However, when one process holds a write lock on a file (or file region), other
  processes are prevented from holding either read or write locks on that file (or file region)
- _Mutexes and condition variables:_ These synchronization facilities are normally used with POSIX
  threads

## Pipes and FIFOs

Pipes can be used to pass data between related processes. FIFOs are a variation on the pipe concept.
Pipes are unidirectional.

![UNIX Pipe](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/unix_pipe.drawio.png)

A pipe is a byte stream, that means there is no concept of messages or message boundaries when using
a pipe. The process reading from a pipe can read blocks of data of any size, regardless of the size
of blocks written by the writing process. Furthermore, the data passes through the pipe sequentially

Attempts to read from a pipe that is currently empty block until at least one byte has been written
to the pipe. If the write end of a pipe is closed, then a process reading from the pipe will see
end-of-file once it has read all remaining data in the pipe

Writes of up to PIPE_BUF bytes are guaranteed to be atomic

A pipe is simply a buffer maintained in kernel memory. This buffer has a maximum capacity. Once a
pipe is full, further writes to the pipe block until the reader removes some data from the pipe

## _popen()_

A common use for pipes is to execute a shell command and either read its output or send it some input.
The _popen()_ and _pclose()_ functions are provided to simplify this task

```c
#include <stdio.h>

FILE *popen(const char *command, const char *mode);

int pclose(FILE *stream);
```

![UNIX Pipe](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/popen.drawio.png)

## FIFOs

Semantically, a FIFO is similar to a pipe. The main difference is that a FIFO has a name within the
file system and is opened in the same way as a regular file. This allows a FIFO to be used for communication
between unrelated processes.

```c
#include <sys/stat.h>

int mkfifo(const char *pathname, mode_t mode);
```

Opening a FIFO synchronizes the reading and writing processes

- Once a FIFO has been created, any process can open it, subject to the usual file permission checks.
- Opening a FIFO for reading blocks until another process opens the FIFO for writing
- Opening the FIFO for writing blocks until another process opens the FIFO for reading