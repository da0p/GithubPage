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

- If threads are not joined, they will become _zombie_ when they terminate 
- Threads are peers. Any thread in a process can join with any other thread in the process. It is
  no such relationship with processes. Only parent process can wait for their children
- There is no way to join with any thread and do nonblocking join. Thread must join with another
  specific thread and the need of using condition variables to achive nonblocking join

## Mutexes

In order to avoid race conditions between threads, _mutex_ can be used in order to ensure that only
one thread at a time can access the variable. More generally, mutexes are used to ensure atomic
access to any shared resource.

A mutex has two states: _locked_ and _unlocked_. At any moment, at most one thread may hold the lock
on a mutex. Attempting to lock a mutex that is already locked either blocks or fails with an error,
depending on the method used to place the lock.

Some notes:

- When a thread locks a mutex, it becomes the owner of that mutex. Only the mutex owner can
unlock the mutex.
- A mutex can either be allocated as a static variable or be created dynamically at run time
- Mutexes are implemented using atomic machine-language operations (performed on memory locations
  visible to all threads) and require system calls only in case of lock contention, therefore it can
  be faster than file locks and semaphores since they always require a system call for the lock and
  lock operations
- Two approaches to avoid a deadlock
  - Define a mutex hierarchy and all threads follow this rule
  - Use try lock mechanism by first locking the first mutex normally, but with the second mutex,
    locking it with try lock mechanism, if it fails, release all mutexes and tries again. This
    approach is less efficient since the threads require to loop

### Mutex Types

| Type                         | Brief Description                                                                                                                                               |
| ---------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| _PTHREAD\_MUTEX\_NORMAL_     | (Self-) deadlock detection is not provided for this type of mutex. Deadlock can happen if a thread tries to lock a mutex that it has already locked             |
|                              | Unlocking a mutex that is not locked or that is locked by another thread procudes undefined results                                                             |
| _PTHREAD\_MUTEX\_ERRORCHECK_ | Error checking is performed on all operations                                                                                                                   |
| _PTHREAD\_MUTEX\_RECURSIVE_  | Maintains the concept of a lock count. When a thread subsequently locks the mutex, the lock count in increased. The mutex is released only when lock count is 0 |

## Condition Variables

Condition variables allow one thread to inform other threads about changes in the state of a shared
variable (or other shared resource) and allows the other threads to wait (block) for such notification.
One advantage of condition variable is to prevent wasting CPU time

Condition variables can be allocated statically or dynamically. The API for condition variables are

```c
#include <pthread.h>

int pthread_cond_signal(pthread_cond_t *cond);
int pthread_cond_broadcast(pthread_cond_t *cond);
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
```

Condition variable and threads have a natual association:

- Thread locks the mutex in preparation for checking the state of the shared variable
- State of the shared variable is checked
- If the shared variable is not in the desired state, then the thread must unlock the mutext before
  it goes to sleep on the condition variable
- When the thread is reawakened because the condition variable has been signaled, the mutex must
  once more be locked, since, typically, the thread then immediately accesses the shared variable

_pthread\_cond\_wait()_ performs the mutex unlocking and locking required in the last two of these
steps. In the third step, releasing the mutex and blocking on the condition variable are performed
atomically.

Remember that

- _pthread\_cond\_wait()_ call must be governed by a _while_ loop rather an an _if_ statement
- The predicate associated with _cond_  must always be checked after _pthread\_cond\_wait()_  returns
  due to the following reason
  - Other threads may be woken up first
  - Designing for loose predicates may be simpler
  - Spurious wake-ups can occur

```c
for (;;) {
    s = pthread_mutex_lock(&mtx);
    if (s != 0) {
        exit(EXIT_FAILURE);
    }

    // Wait for a condition in a loop
    while (avail == 0) {
        s = pthread_cond_wait(&cond, &mtx);
        if (s != 0) {
            exit(EXIT_FAILURE);
        }
    }

    // Check again the predicate
    while (avail > 0) {
        // do something
        avail--;
    }

    s = pthread_mutex_unlock(&mtx);
    if (s != 0) {
        exit(EXIT_FAILURE);
    }
}
```

Some other notes:

- _pthread\_cond\_signal()_ guarantees that at least one of the blocked threads is woken up, usually
  designed for threads performing the same task
- _pthread\_cond\_broadcast()_ simply wakes up all blocked threads. Usually designed for threads
  performing different tasks
- A condition variable holds no state information. It is simply a mechanism for communicating
  information about the application's state. If no thread is waiting on the condition variable at
  the time that it is signaled, then the signal is lost. A thread that later waits on the condition
  variable will unblock only when the variable is signaled once more