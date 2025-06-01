---
title: "Linux Series: Alternative I/O Models"
date: 2025-06-01
---

## Overview

Traditional blocking I/O model is sufficient for many applications, but not all. In particular, some
applications need to be able to do one or both of the following:

- Check whether I/O is possible on a file descriptor without blocking if it is not possible
- Monitor multiple file descriptors to see if I/O is possible on any of them

_Nonblocking I/O_ allows us to periodically check whether I/O is possible on a file descriptor. If we
need to monitor multiple file descriptors, then we mark them all nonblocking, and poll each of them
in turn. However, polling in this manner is usually undesirable. If polling is done only infrequenly,
then the latency before an application responds to an I/O even may be unacceptably long; on the other
hand, polling in a tight loop wastes CPU time

_Multiple processes_ will allow the parent process to carry on to perform other tasks, while the child
process blocks until the I/O is complete. If we need to handle I/O on multiple file descriptors, we
can create one child for each descriptor. The problems with this approach are expense and complexity.
Creating and maintaining processes places a load on the system, and, typically, the child processes
will need to use some form of IPC to inform the parent about the status of I/O operations

_Multiple threads_ is less demanding of resources, but the threads will probably still need to communicate
information to one another about the status of I/O operations, and the programming can be complex,
especially if we are using thread pools to minimize the number of threads used to handle large numbers
of simultaneous clients

Therefore, one of the following alternatives is often preferable:

- _I/O multiplexing_ allows a process to simultaneously monitor multiple file descriptors to find
  out whether I/O is possible on any of them. The _select()_ and _poll()_ system calls perform I/O
  multiplexing
- _Signal-driven I/O_ is a technique whereby a process requests that the kernel send it a signal
  when input is available or data can be written on a specified file descriptor. The process can then
  carry on performing other activities, and is notified when I/O becomes possible via receipt of the
  signal. When monitoring large numbers of file descriptors, signal-driven I/O provides significantly
  better performance than _select()_ and _poll()_
- _epoll_ API allows a process to monitor mulitple file descriptors to see if I/O is possible on any
  of them. Like signal-drive I/O, the _epoll_ API provides much better performance when monitoring
  large numbers of file descriptors

Compared to signal-driven I/O, _epoll_ provides a number of advantages:

- We avoid the complexities of dealing with signals
- We can specify the kind of monitoring that we want to perform (e.g., ready for reading or ready for
  writing)
- We can select either level-triggered or edge-triggered notification

## Level-Triggered and Edge-Triggered Notification

There are two models of readiness notification for a file descriptor:

- _Level-triggered notification:_ A file descriptor is considered to be ready if it is possible to
  perform an I/O system call without blocking
- _Edge-triggered notification:_ Notification is provided if there is I/O activity on a file descriptor
  since it was last monitored

| **I/O model**        | **Level-triggered** | **Edge-triggered** |
| -------------------- | ------------------- | ------------------ |
| _select()_, _poll()_ | y                   | n                  |
| Signal-driven I/O    | n                   | y                  |
| _epoll_              | y                   | y                  |

When level-triggered notification is used, the readiness of a file descriptor can be checked at any
time. There is no need to perform as much I/O as possible on the file descriptor each time we are
notified that a file descriptor is ready. By contrast, when edge-triggered notification is employed,
we receive notification only when an I/O event occurs. We don't receive any further notification until
another I/O event occurs. Furthermore, when an I/O event is notified for a file descriptor, we usually
don't know how much I/O is possible. Therefore, programs that employ edge-triggered notification are
usually designed:

- Perform as much I/O as possible on the file descriptor when got an notification
- Use non-blocking mode and perform I/O operations repeatedly until the relevant system call fails
  with the error _EAGAIN_ or _EWOULDBLOCK_

## I/O Multiplexing

I/O multiplexing allows us to simultaneously monitor multiple file descriptors to see if I/O is possible
on any of them. We can perform I/O multiplexing using either of two system calls with essentially the
same functionality:

-_select()_
-_poll()_

We can use _select()_ and _poll()_ to monitor file descriptors for regular files, terminals, pseudoterminals,
pipes, FIFOs, sockets, and some types of character devices

## The _select()_ System Call

```c
#include <sys/time.h>
#include <sys/select.h>

int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

- _readfds_ is the set of file descriptors to be tested to see if input is possible
- _writefds_ is the set of file descriptors to be tested to see if output is possible
- _exceptfds_ is the set of file descriptors to be tested to see if an exceptional condition has occurred

_exceptional condition_ means:

- A state change occurs on a pseudoterminal slave connected to a master that is in packet mode
- Out-of-band data is received on a stream socket

_select()_ returns one of the following:

- A return value of -1 indicates that an error occurred. Possible errors include _EBADF_ and _EINTR_.
  _EBADF_ indicates that one of the file descriptors in _readfds_, _writefds_, or _exceptfds_ is invalid.
  _EINTR_ indicates that the call was interrupted by a signal handler
- A return value of 0 means that the call timed out before any file descriptor became ready. In this
  case, each of the returned file descriptor sets will be empty
- A positive return value indicates that one or more file descriptors is ready. The return value is
  the number of ready descriptors. In this case, each of the returned file descriptor sets must be
  examined (using _FD\_ISSET()_) in order to find out which I/O events occurred. If the same file
  descriptor is specified in more than one of _readfds_, _writefds_, and _exceptfds_, it is counted
  multiple times if it is ready for more than one event. In other words, _select()_ returns the total
  number of file descriptors marked as ready in all three returned sets

## The _poll()_ System Call

The _poll()_ system call performs a similar task to _select()_. The major difference between the two
system calls lies in how we specify the file descriptors to be monitored. With _select()_, we provide
three sets, each marked to indicate the file descriptors of interest. With _poll()_, we provide a list
of file descriptors, each marked with the set of events of interest

```c
#include <poll.h>

int poll(struct pollfd fds[], nfds_t nfds, int timeout);
```

### The _pollfd_ array

The _fds_ argument lists the file descriptors to be monitored by _poll()_. This argument is an array
of _pollfd_ structures, defined as follows:

```c
struct pollfd {
    int fd;             // file descriptor
    short events;       // requested events bit mask
    short revents;      // returned events bit mask
};
```

_nfds_ specifies the number of items in the _fds_ array. The _events_ and _revents_ fields of the _pollfd_
structure are bit masks. The caller initializes _events_ to specify the events to be monitored for the
file descriptor _fd_. Upon return from _poll()_, _revents_ is set to indicate which of those events
actually occurred for this file descriptor

| Bit        | Input in _events?_ | Returned in _revents?_ | Description                                    |
| ---------- | ------------------ | ---------------------- | ---------------------------------------------- |
| POLLIN     | y                  | y                      | Data other than high-priority data can be read |
| POLLRDNORM | y                  | y                      | Equivalent to POLLIN                           |
| POLLRDBAND | y                  | y                      | Priority data can be read (unused on Linux)    |
| POLLRPI    | y                  | y                      | High-priority data can be read                 |
| POLLRDHUP  | y                  | y                      | Shutdown on peer socket                        |
| POLLOUT    | y                  | y                      | Normal data can be written                     |
| POLLWRNORM | y                  | y                      | Equivalent to POLLOUT                          |
| POLLWRBAND | y                  | y                      | Priority data can be written                   |
| POLLERR    | n                  | y                      | An error has occurred                          |
| POLLHUP    | n                  | y                      | A hangup has occurred                          |
| POLLNVAL   | n                  | y                      | File descriptor is not open                    |
| POLLMSG    | n                  | n                      | Unused on Linux                                |

_poll()_ returns one of the following:

- A return value of -1 indicates that an error occurred. One possible error is _EINTR_, indicating that
  the call was interrupted by a signal handler
- A return of 0 means that the call timed out before any file descriptor became ready
- A positive return value indicates that one or more file descriptors are ready. The returned value
  is the number of _pollfd_ structures in the _fds_ array that have a nonzero _revents_ field

## Readiness of a File Descriptor (using _select()_ or _poll()_)

A file descriptor (with _O\_NONBLOCK_ clear) is considered to be ready if a call to an I/O function
would not block, regardless of whether the function would actually transfer data. That means _select()_
and _poll()_ tell us whether an I/O operation would not block, rather than whether it would successfully
transfer data

### Regular files

File descriptors that refer to regular files are always marked as readable and writable by _select()_,
and returned with _POLLIN_ and _POLLOUT_ set in _revents_ for _poll()_, for the following reason:

- A _read()_ will always immediately return data, end-of-file, or an error
- A _write()_ will always immediately transfer data or fail with some error

### Terminals and pseudoterminals

| Condition or event                                              | _select()_ | _poll()_                                                               |
| --------------------------------------------------------------- | ---------- | ---------------------------------------------------------------------- |
| Input available                                                 | r          | POLLIN                                                                 |
| Output possible                                                 | w          | POLLOUT                                                                |
| After _close()_ by pseudoterminal peer                          | rw         | POLLHUP on Linux; POLLHUP, POLLERR, or POLLIN on other implementations |
| Pseudoterminal master in packet mode detects slave state change | x          | POLLRPI                                                                |

### Pipes and FIFOs

Assume that _POLLIN_ was specified in the _events_ field for _poll()_

_select()_ and _poll()_  indications for the read end of a pipe or FIFO

| Data in pipe? | Write end open? | _select()_ | _poll()_            |
| ------------- | --------------- | ---------- | ------------------- |
| no            | no              | r          | POLLHUP             |
| yes           | yes             | r          | POLLIN              |
| yes           | no              | r          | POLLIN   or POLLHUP |


Assume that _POLLOUT_ was specified in the _events_ field for _poll()_

_select()_ and _poll()_ indications for the write end of a pipe or FIFO

| Space for PIPE_BUF bytes? | Read end open? | _select()_ | _poll()_           |
| ------------------------- | -------------- | ---------- | ------------------ |
| no                        | no             | w          | POLLERR            |
| yes                       | yes            | w          | POLLOUT            |
| yes                       | no             | w          | POLLOUT or POLLERR |

### Sockets

Assume that _events_ was specified as (POLLIN | POLLOUT | POLLRPI) for _poll()_
Assume that the file descriptor is being tested to see if input is possible, output is possible, or
an exceptional condition occurred

_select()_ and _poll()_ indications for sockets

| Condition or event                                                         | _select()_ | _poll()_                       |
| -------------------------------------------------------------------------- | ---------- | ------------------------------ |
| Input available                                                            | r          | POLLIN                         |
| Output possible                                                            | w          | POLLOUT                        |
| Incoming connection established on listening socket                        | r          | POLLIN                         |
| Out-of-band data received (TCP only)                                       | x          | POLLPRI                        |
| Stream socket peer closed connection or executed _shutdown()_ (_SHUT\_WR_) | rw         | POLLIN or POLLOUT or POLLRDHUP |

### Problems with _select()_ and _poll()_

- On each call to _select()_ or _poll()_, the kernel must check all of the specified file descriptors
  to see if they are ready. When monitoring a large number of file descriptors that are in a densely
  packaged range, the time required for this operation greatly outweighs the time required for the
  next two operations
- Copying data back and forth between user space and kernel space consumes a noticeable amount of CPU
  time when monitoring many file descriptors. Besides, the size of the data structure for _select()_
  is fixed by _FD\_SETSIZE_, regardless of the number of file descriptors being monitored
- After the call to _select()_ or _poll()_, the program must inspect every element of the returned data
  structure to see which file descriptors are ready

Signal-drive I/O and _epoll_ are both mechanisms that allow the kernel to record a persistent list of
file descriptors in which a process is interested. Doing this eliminates the performance scaling problems
of _select()_ and _poll()_, yielding solutions that scale according to the number of I/O events that
occur, rather than according to the number of file descriptors being monitored. Consequently, signal-drive
I/O and _epoll_ provide superior performance when monitoring large numbers of file descriptors

## Signal-Driven I/O

With signal-drive I/O, a process requests that the kernel send it a signal when I/O is possible on a
file descriptor. The process can then perform any other activity until I/O is possible, at which time
the signal is delivered to the process. To use signal-drive I/O, a program performs the following steps:

- Establish a handler for the signal delivered by the signal-drive I/O mechanism. By default, this
  notification signal is _SIGIO_, otherwise _SIGIO_ will terminate the running process
- Set the _owner_ of the file descriptor - that is, the process or process group that is to receive
  signals when I/O is possible on the file descriptor. Typically, we make the calling process the owner.
- Enable nonblocking I/O by setting the _O\_NONBLOCK_ open file status flag using _fcntl()_
- Enable signal-driven I/O by turning on the _O\_ASYNC_ open file status flag
- The calling process can now perform other tasks. When I/O becomes possible, the kernel generates
  a signal for the process and invokes the signal handler established in the first step
- Signal-drive I/O provides edge-triggered notification. This means that once the process has been
  notified that I/O is possible, it should perform as much I/O as possible. Assuming a non-blocking
  file descriptor, this means executing a loop that performs I/O system calls until a call fails with
  the error _EAGAIN_ or _EWOULDBLOCK_

## Readiness of I/O Possible Signal

I/O possible is signaled for various file types

### Terminals and pseudoterminals

For terminals and pseudoterminals, a signal is generated whenever new input becomes available, even
if previous input has not yet been read. Input-possible is also signaled if an end-of-file condition
occurs on a terminal (but not on a pseudoterminal)

No output-possible signaling for terminals

### Pipes and FIFOs

Read end of a pipe or FIFO:

- Data is written to the pipe (even if there was already unread input available)
- Write end of the pipe is closed

Write end of a pipe or FIFO:

- A read from the pipe increases the amount of free space in the pipe so that it is now possible to
  write _PIPE\_BUF_ bytes without blocking
- Read end of the pipe is closed

### Sockets

Signal-driven I/O works for datagram sockets in both the UNIX and the Internet domains. A signal is
generated in the following circumstances:

- An input datagram arrives on the socket (even if there were already unread datagrams waiting to be read)
- An asynchronous error occurs on the socket

Signal-driven I?O works for stream sockerts in both UNIX and Internet domains. A signal is generated
in the following circumstances:

- A new connection is received on a listening socket
- A TCP _connect()_ request completes; that is, the active end of a TCP connection entered the _ESTABLISHED_
  state. The analogous condition is not signaled for UNIX domain sockets
- New input is received on the socket (even if there was already unread input available)
- The peer closes its writing half of the connection using _shutdown()_, or closes its socket altogether
  using _close()_
- Output is possible ont he socket
- An asynchronous error occurs on the socket

## The _epoll_ API

The Linux _epoll_ (event poll) API is used to monitor multiple file descriptors to see if they are ready
for I/O. The primary advantages of the _epoll_ API are the following:

- The performance of _epoll_ scales much better than _select()_ and _poll()_ when monitoring large numbers
  of file descriptors
- The _epoll_ API permits either level-triggered or edge-triggered notification. By contrast, _select()_
  and _poll()_ provide only level-triggered notification, and signal-driven I/O provides only edge-triggered
  notification

The performance of _epoll_ and signal-drive I/O is similar. However, _epoll_ has some advantages over
signal-driven I/O:

- We avoid the complexities of signal handling (e.g., signal-queue overflow)
- We have greater flexibility in specifying what kind of monitoring we want to perform (e.g., checking
  to see if a file descriptor for a socket is ready for reading, writing, or both)

The central data structure of the _epoll_ API is an _epoll instance_, which is referred to via an open
file descriptor. This file descriptor is not used for I/O. Instead, it is a handle for  kernel data
structures that serve two purposes:

- recording a list of file descriptors that this process has declared an interest in monitoring - the
  _interest list_
- maintaining a list of file descriptors that are ready for I/O - the _ready list_

The membership of the ready list is a subset of the interest list

For each file descriptor monitored by _epoll_, we can specify a bit mask indicating events that we are
interested in knowing about. These bit masks correspond closely to the bit masks used with _poll()_

The _epoll_ API consists of three system calls:

- The _epoll\_create()_ system call creates an _epoll_ instance and returns a file descriptor referring
  to the instance
- The _epoll\_ctl()_ system call manipulates the interest list associated with an _epoll_ instance.
  Using _epoll\_ctl()_, we can add a new file descriptor to the list, remove an existing descriptor
  from the list, and modify the mask that determines which events are to be monitored for a descriptor
- The _epoll\_wait()_ system call returns items from the ready list associated with an _epoll_ instance

Some notes:

- Each file descriptor registered in an _epoll_ interest list requires a small amount of nonswappable
  kernel memory, the kernel provides an interface that defines a limit on the total number of file
  descriptors that each user can register in an _epoll_ interest lists. The value of this limit can
  be viewed and modified via _max\_user\_watches_
- In a multithreaded program, it is possible for one thread to use _epoll\_ctl()_ to add file descriptors
  to the interest list of an _epoll_ instance that is already being monitored by _epoll\_wait()_ in
  another thread. These changes to the interest list will be taken into account immediately, and the
  _epoll\_wait()_ call will return readiness information about the newly added file descriptors

### _epoll_ events

The bit values that can be specified in _ev.events_ when we call _epoll\_ctl()_ and that are placed
in the _evlist[].events_ fields returned by _epoll\_wait()_ are shown below

| Bit          | Input to _epoll\_ctl()_ | Returned by _epoll\_wait()_? | Description                                    |
| ------------ | ----------------------- | ---------------------------- | ---------------------------------------------- |
| EPOLLIN      | y                       | y                            | Data other than high-priority data can be read |
| EPOLLPRI     | y                       | y                            | High-priority data can be read                 |
| EPOLLRDHUP   | y                       | y                            | Shutdown on peer socket                        |
| EPOLLOUT     | y                       | y                            | Normal data can be written                     |
| EPOLLET      | y                       | n                            | Employ edge-triggered event nofitication       |
| EPOLLONESHOT | y                       | n                            | Disable monitoring after event notification    |
| EPOLLERR     | n                       | y                            | An error has occurred                          |
| EPOLLHUP     | n                       | y                            | A hangup has occurred                          |

### Performance Comparison of _epoll_ and I/O Multiplexing 

- On each call to _select()_ or _poll()_, the kernel must check all of the file descriptors specified
  in the call. By contrast, when we mark a descriptor to be monitored with _epoll\_ctl()_, the kernel
  records this fact in a list associated with the underlying open file description, and whenever an I/O
  operation that makes the file descriptor ready is performed, the kernel adds an item to the ready list
  for the _epoll_ descriptor. (An I/O event on a single open file description may cause multiple file
  descriptors associated with that description to become ready.) Subsequent _epoll\_wait()_ calls simply
  fetch items from the ready list
- Each time we call _select()_ or _poll()_, we pass a data structure to the kernel that identifies all
  of the file descriptors that are to be monitored, and, on return, the kernel passes back a data structure
  describing the readiness of all of these descriptors. By contrast, with _epoll_, we use _epoll\_ctl()_
  to build up a data structure in kernel space that lists the set of file descriptors to be monitored.
  Once this data structure has been built, each later call to _epoll\_wait()_ doesn't need to pass any
  information about file descriptors to the kernel, and the call returns information about only those
  descriptors that are ready

### Edge-Triggered Notification

By default, the _epoll_ mechanism provides _level-triggered_ notification. The _epoll_ API also allows
for _edge-triggered_ notification - that is, a call to _epoll\_wait()_ tells us if there has been I/O
activity on a file descriptor since the previous call to _epoll\_wait()_ (or since the descriptor was
opened, if there was no previous call). Using _epoll_ with edge-triggered notification is semantically
similar to signal-driven I/O, except that if multiple I/O events occur, _epoll_ coalesces them into a
single notification returned via _epoll\_wait()_; with signal-driven I/O, multiple signals may be generated