---
title: "Linux Series: File Locking"
date: 2025-05-27
---

## Overview

File locking is used to avoid race condition of two or more processes performing actions on a file. Even
though we could use semaphores to perform the required synchronization, using file locks is usually
preferable, because the kernel automatically associates locks with files.

Two different APIs for placing the file locks:

- _flock()_, which places locks on entire files
- _fcntl()_, which places locks on regions of a file

The general method of using _flock()_ and _fcntl()_ is as follows:

- Place a lock on the file
- Perform file I/O
- Unlock the file so that another process can lock it

In order to mix locking and _stdio_ functions, we need to be cautious because of the user-space buffering
performed by the _stdio_ library. The problem is that an input buffer might be filled before a lock
is placed, or an output buffer may be flushed after a lock is removed. There are a few ways to avoid
these problems:

- Perform file I/O using _read()_ and _write()_ instead of _stdio_ library
- Flush the _stdio_ stream immediately after placing a lock on the file, and flush it once more immediately
  before releasing the lock
- Perharps at the cost of some efficiency, disable _stdio_ buffering altogether using _setbuf()_

## File Locking with _flock()_

The _flock() system call places a single lock on an entire file

```c
#include <sys/file.h>

int flock(int fd, int operation);
```

| Value   | Description                                               |
| ------- | --------------------------------------------------------- |
| LOCK_SH | Place a _shared_ lock on the file referred to by _fd_     |
| LOCK_EX | Place an _exclusive_ lock on the file referred to by _fd_ |
| LOCK_UN | Unlock the file referred to by _fd_                       |
| LOCK_NB | Make a nonblocking lock request                           |

Any number of processes may simultaneously hold a shared lock on a file. However, only one process at
a time can hold an exclusive lock on a file

| Process A | Process B |         |
| --------- | --------- | ------- |
|           | LOCK_SH   | LOCK_EX |
| LOCK_SH   | Yes       | No      |
| LOCK_EX   | No        | No      |

A process can place a shared or exclusive lock regardless of the access mode (read, write, or
read-write) of the file. An existing shared lock can be converted to an exclusive lock (and vice versa)
by making another call to _flock()_ specifying the appropriate value for _operation_.
Converting a shared lock to an exclusive lock will block if another process holds a shared lock on the
file, unless LOCK_NB was also specified. A lock conversion is _not_ guaranteed to be atomic

## Record Lockng with _fcntl()_

Using _fcntl()_, we can place a lock on any part of a file, ranging froma  single byte to the entire
file. This form of file locking is usually called _record locking_. Typically, _fcntl()_ is used to lock
byte ranges corresponding to the application-defined record boundaries within the file; hence the origin
of the term _record locking_.

The _flock_ structure defines the lock that we wish to acquire or remove

```c
struct flock {
    short l_type; /* Lock type: F_RDLCK, F_WRLCK, F_UNLCK */
    short l_whence; /* How to interpret 'l_start': SEEK_SET, SEEK_CUR, SEEK_END */
    off_t l_start; /* Offset where the lock begins */
    off_t l_len; /* Number of bytes to lock; 0 means "until EOF"*/
    pid_t l_pid; /* Process preventing our lock (F_GETLK only) */
}
```

| Lock type | Description             |
| --------- | ----------------------- |
| F_RDLCK   | Place a read lock       |
| F_WRLCK   | Place a write lock      |
| F_UNLCK   | Remove an existing lock |

![Record Locking](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/record_locking.drawio.png)

In order to place a read lock on a file, the file must be open for reading. Similarly, to place a write
lock, the file must be open for writing. To place both types of locks, we open the file read-write (O_RDWR).
Attempting to place a lock that is incompatible with the file access mode results in the error _EBADF_

## Lock Acquisition and Release

- Unlocking a file region always immediately succeeds. It is not an error to unlock a region on which
  we don't currently hold a lock
- At any time, a process can hold just one type of lock on a particular region of a file. Placing a
  new lock on a region we have already locked either results in no change (if the lock type if the
  same as the existing lock) or automatically converts the existing lock to the new mode.
- A process can never lock itself out of a file region, even when placing locks via multiple file
  descriptors referring to the same file
- Placing a lock of a different mode in the middle of a lock we already hold results in three locks:
  two smaller locks in the previous mode are created on either side of the new lock
- Closing a file descriptor has some unusual semantics with respect to file region locks
- Deadlock situations are detected by the kernel. The kernel checks each new lock request made via F_SETLKW
  to see if it would result in a deadlock situation. If it would, then the kernel selects one of the
  blocked processes and causes its _fcntl()_ call to unblock and fail witht he error _EDEADLK_

![Lock Splitting](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/lock_splitting.drawio.png)

## Lock Inheritance and Release

- Record locks are not inherited across a _fork()_ by a child process. This contrasts with _flock()_,
  where the child inherits a reference to the _same_ lock and can release this lock, with the consequence
  that the parent also loses the lock
- Record locks are preserved across an _exec()_
- All of the threads in a process share the same set of record locks
- Record locks are associated with both a process and an i-node. The consequence of this association
  is that
  - when a process terminates, all of its record locks are released
  - whenever a process closes a file descriptor, all locks held by the process on the corresponding
    file are released, regardless of the file descriptor through which the locks were obtained