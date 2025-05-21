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

### _popen()_

A common use for pipes is to execute a shell command and either read its output or send it some input.
The _popen()_ and _pclose()_ functions are provided to simplify this task

```c
#include <stdio.h>

FILE *popen(const char *command, const char *mode);

int pclose(FILE *stream);
```

![UNIX Pipe](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/popen.drawio.png)

### FIFOs

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

## POSIX IPC

Three POSIX IPC mechanisms are the following:

- _Message queues_ can be used to pass messages between processes. Message boundaries are preserved,
  so that readers and writers communicate in units of messages. POSIX message queues permit each
  message to be assigned a priority, which allows high-priority messages to be queued ahead of low-priority
  messages
- _Semaphores_ permit multiple processes to synchronize their actions. POSIX semaphore is a kernel-maintained
  integer whose value is never permitted to go below 0. POSIX semaphores are allocated individually and
  operated individually using two operations that increase and decrease a semaphore's value by one
- _Shared memory_ enables multiple processes to share the same region of memory. Once one process has
  updated the shared memory, the change is immediately visible to other processes sharing the same
  region

## POSIX Message Queues

The main functions in the POSIX message queue API are the following:

- The _mq\_open()_ function creates a new message queue or opens an existing queue, returning a message
  queue descriptor for use in later calls
- The _mq\_send()_ function writes a message to a queue
- The _mq\_receive()_ function reads a message from a queue
- The _mq\_close()_ function closes a message queue that the process previously opened
- The _mq\_unlink()_ function removes a message queue name and marks the queue for deletion when all
  processes have closed it
- _mq\_setattr()_ and _mq\_getattr()_ can be used to change a message queue attributes or get the
  message queue attributes
- _mq\_notify()_ function allows a process to register for message notification from a queue. After
  registering, the process is notified of the availability of a message by delivery of a signal or by
  the invocation of a function in a separate thread

### Relationship Between Descriptors and Message Queues

The relationship between a message queue descriptor and an open message queue is analogous to the
relationship between a file descriptor and open file. A message queue descriptor is a per-process
handle that refers to an entry in the system-wide table of open message queue descriptions, and this
entry in turn refers to a message queue object.

![Message Queue Descriptor Table](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/message_queue_table.drawio.png)

### Message Notification

A feature that distinguishes POSIX message queues from their System V counterparts is the ability to
receive asynchronous notification of the availability of a message on a previously empty queue. This
feature means that instead of making a blocking _mq\_receive()_ call or making the message queue
descriptor nonblocking and performing periodic _mq\_receive()_ calls on the queue, a process can request
a notification of message arrival and then perform other tasks until it is notified. A process can
choose to be notified via a signal or via invocation of a function in a separate thread

```c
#include <mqueue.h>

int mq_notify(mqd_t mqdes, const struct sigevent *notification);
```

There are a few points to be noted

- At any time, only one process can be registered to receive a notification from a particular message
  queue. If there is already a process registered for a message queue, further attempts to register
  for that queue fail (_mq\_notify()_ fails with the error _EBUSY_)
- The registered process is notified only when a message arrives on a queue that was previously empty.
  If a queue already contains messages at the time of the registration, a notification will occur only
  after the queue is emptied and a new message arrives
- After one notification is sent to the registered process, the registration is removed, and any process
  can then register itself for notification. In other words, as long as a process wishes to keep
  receiving notifications, it must reregister itself after each notification by once again calling _mq\_notify()_
- The registered process is notified only if some other process is not currently blocked in a call to
  _mq\_receive()_ for the queue. If some other process is blocked in _mq\_receive()_, that process will
  read the message, and the registered process will remain registered
- A process can explicitly deregister itself as the target for message notification by calling _mq\_notify()_
  with a _notification_ argument of NULL

## POSIX Semaphores

Two types of POSIX semaphores:

- _Named semaphores:_ This type of semaphore has a name. By calling _sem\_open()_ with the same name,
  unrelated processes can access the same semaphore
- _Unnamed semaphores:_ This type of semaphore doesn't have a name; instead, it resides at an
  agreed-upon location in memory. Unnamed semaphores can be shared between processes or between a
  group of threads. When shared between processes, the semaphore must reside in a region of shared
  memory. When shared between threads, the semaphore may reside in an area of memory shared by the
  threads (e.g., on the heap or in a global variable)

### Named Semaphores

The following functions are needed to work with a named semaphore:

- The _sem\_open()_ function opens or creates a semaphore, initializes the semaphore if it is created
  by the call, and returns a handle for use in later calls
- The _sem\_post(sem)_ and _sem\_wait(sem)_ functions respectively increment and decrement a semaphore's
  value
- The _sem\_getvalue()_ function retrieves a semaphore's current value
- The _sem\_close()_ function removes the calling process's association with a semaphore that it previously
  opened
- The _sem\_unlink()_ function removes a semaphore name and marks the semaphore for deletion when all
  processes have closed it

### Unnamed Semaphores

Unnamed semaphores (also known as _memory\-based semaphores_) are variables of type _sem\_t_ that are
stored in memory allocated by the application. The semaphore is made available to the processes or
threads that use it by placing it in an area of memory that they share

Operations on unnamed semaphores use the same functions (_sem\_wait()_, _sem\_post()_, _sem\_getvalue()_,
and so on) that are used to operate on named semaphores. In addition, two further functions are required:

- The _sem\_init()_ function initializes a semaphore and informs the system of whether the semaphore
  will be shared between processes or between the threads of a single process
- The _sem\_destroy(sem)_ function destroys a semaphore

Unamed semaphore that is usually used when

- Shared between threads, there is no need for a name
- Shared between related processes (e.g. parent and child)