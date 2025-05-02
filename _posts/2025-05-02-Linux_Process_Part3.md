---
title: "Linux Series: Monitoring Child Processes"
date: 2025-05-02
---

A parent process needs to know when one of its child processes changes state - when the child terminates
or is stopped by a signal

## Waiting on a Child Process

Linux provides facility to monitor child processes by using _wait()_ or _waitpid()_ system calls

```c
#include <sys/wait.h>

pid_t wait(int *status);
```

### _wait()_ System Call

**_wait()_** system call does the following things:

- If no child of the calling process has yet terminated, the call blocks until one of the children
  terminates. If a child has already terminated by the time of the call, _wait()_ returns immediately 
- If _status_ is not NULL, information about how the child terminated is returned in the integer to
  which _status_ points
- The kernel adds the process CPU times and resource usage statistics to running totals for all
  children of this parent process
- As its function result, _wait()_ returns the process ID of the child that has terminated

On error, _wait()_ returns -1. One possible error is that the calling process has no children, which
is indicated by the _errno_ value _ECHILD_

It has numerous limitations:

- If a parent process has created multiple children, it is not possible to _wait()_ for the completion
  of a specific child. We can only wait for the next child that terminates
- If no child has yet terminated, _wait()_ always blocks. Sometimes, it would be preferable to perform
  a nonblocking wait so that if no child has yet terminated, we obtain an immediate indication of this
  fact
- Using _wait()_, we can find out only about children that have terminated. It is not possible to be
  notified when a child is stopped by a signal (such as _SIGSTOP_ or _SIGTTIN_) or when a stopped child
  is resumed by delivery of a _SIGCONT_ signal

### _waitpid() System Call

```c
#include <sys/wait.h>

pid_t waitpid(pid_t pid, int *status, int options);
```

**_waitpid()_** system call does the following things to overcome the limitations of _wait()_:

- If _pid_ is greater than 0, wait for the child whose _process ID_  equals _pid_
- If _pid_ equals 0, wait for any child in the _same process group as the caller_ (parent)
- If _pid_ less than -1, wait for any child whose _process group_ identifier equals the absolute
  value of _pid_
- If _pid_ equals -1, wait for _any_ child

## Orphans and Zombies

The lifetime of parent and child processes are usually not the same - either the parent outlives the
child or vice versa. There are two cases:

- When the child process outlives the parent process, the orphaned child is
adopted by _init_, the ancestor of all processes, whose process ID is 1

- When the child process terminates before its parent process has had no chance to perform a _wait().
  The child process becomes a _zombie_. A _zombie_ can't be killed by a signal, even _SIGKILL_. This
  ensures that the parent can always eventually perform a _wait()_. When the parent does perform a
  _wait()_, the kernel removes the zombie, since the last remaining information about the child is no
  longer required. On the other hand, if the parent terminates without doing a _wait()_, then the
  _init_ process adopts the child and automatically performs a _wait()_, thus removing the zombie
  process from the system. The problem happens when the parent fails to perform a _wait()_, then the
  entry for zombie child will be maintained indefinitely in the kernel's process table. The only way
  to kill those zombies is to kill their parents and let _init_ process adopts them, then perform _wait()_

## The _SIGCHLD_ Signal

The termination of a child process is an event that occurs asynchronously. A parent can't predict
when one of its child will terminate. Even if the parent sends a _SIGKILL_ signal to the child, the
exact time of termination is still dependent on when the child is next scheduled for use of a CPU.
There are several solutions:

- Parent process can call _wait()_, or _waitpid()_ without specifying the _WNOHANG_ flag, in which
  case the call will block if a child has not already terminated
- Parent can periodically perform a nonblocking check (a poll) for dead children via a call to
  _waitpid()_ specifying the _WNOHANG_ flag
- Or employ handler for _SIGCHLD_ signal to avoid wasting CPU time

In order to establish a handler for _SIGCHLD_ in this case, note that:

- _SIGCHLD_ is not queued, so if a parent has multiple child processes, if child processes terminate
  close to each other, then parent process receives only one _SIGCHLD_ event. The solution is to loop
  inside the handler to reap all dead children with _WNOHANG_ flag

```c
while (waitpid(-1, nullptr, WNOHANG) > 0) {
    continue;
}
```

Note that for stopped children, it is possible for a parent process to receive the _SIGCHLD_ signal
when one of its children is stopped by a signal. This behavior is controlled by the _SA\_NOCLDSTOP_
flag when using _sigaction()_ to establish a handler for the _SIGCHLD_ signal