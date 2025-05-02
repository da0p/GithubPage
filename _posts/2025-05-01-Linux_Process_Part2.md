---
title: "Linux Series: Processes Management"
date: 2025-05-01
---

## Process Creation

There are several important system calls concerning the topic of creating and terminating processes:

- _fork()_ system call allows one process, the parent, to create a new process, the child. This is
done by making the new child process an (almost) exact duplicate of the parent: the child process
obtains copies of the parent's stack, data, heap, and text segments
- _exit(status)_  terminates a process, making all resources (memory, open file descriptors, and so
on) used by the process available for subsequent reallocation by the kernel
- _wait(status)_ has two purposes. **First**, if a child of this process has not yet terminated by
calling _exit()_, then _wait()_ suspends execution of the process until one of its children has
terminated. **Second**, the termination status of the child is returned in the status argument of _wait()_
- _execve(pathname, argv, envp)_ loads a new program (_pathname_, with argument list _argv_, and
environment list _envp_) into a process's memory. The existing program text is discarded, and the
stak, data, and heap segments are freshly created for the new program. This operation is often
referred to as _execing_ a new program

![Overview](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/overview_procress_creation.drawio.png)

### File Sharing Between Parent and Child

When a _fork()_ is performed, the child receives duplicates of all of the parent's file descriptors.
These duplicates are made in the manner of _dup()_, which means that corresponding descriptors in
the parent and the child refer to the same open file description. The open file description contains
the current file offset and the open file status flags. Consequently, these attributes of an open
file are shared between parent and child. That means, if the child updates the file offset, this
change is visible through the corresponding descriptor in the parent.

If sharing of file descriptors in this manner is not required, then an application should be designed
so that, after a _fork()_, the parent and child use different file descriptors, with each process
closing unused descriptors immediately after forking

![File descriptor sharing](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/dup_fd_during_fork.drawio.png)

### Memory Semantics of _fork()_

Performing a simple copy of the parent's virtual memnory pages into the new child process would be
wasteful for the following reasons:

- _fork()_ is often followed by an immediate _exec()_, which replaces the process's text with a new
program and reinitializes the process's data, heap, and stack segments. Most modern UNIX
implementations, including Linux use two techniques to avoid such wasteful copying
    - The kernel marks the text segment of each process as read-only, so that a process can't modify
    its own code. This means that the parent and child can share the same text segment. The _fork()_
    system call creates a text segment for the child by building a set of per-process page-table
    entries that refer to the same virtual memory page frames already used by the parent
    - For the pages in the data, heap, and stack segments of the parent process, the kernel employs
    a technique known as _copy\-on\-write_

![Copy on Write](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/copy_on_write.drawio.png)

### Race Conditions

After a _fork()_, it is undetermined if the parent process or child process will run first. This
situation can cause race conditions. In order to avoid such problem, we can rely on signals to
synchronize the parent and child process

## Process Termination

A process may terminate in two general ways:

- **Abnormal termination**: caused by the delivery of a signal whose default action is to terminate
  the process
- **Normal termination**: by using _\_exit()_ system call

During both normal and abnormal termination of a process, the following actions occur:

- Open file descriptors, directory streams, message catalog descriptors, and conversion descriptors
  are closed
- As a consequence of closing file descriptors, any file locks held by this process are released
- Any attached System V shared memory segments are detached, and the _shm\_nattch_ counter
  corresponding to each segment is decremented by one
- For each System V semaphore for which a _semadj_ value has been set by the process, that _semadj_
  value is added to the semaphore value
- If this is the controlling process for a controlling terminal, then the _SIGHUP_ signal is sent to
  each process in the controlling terminal's forground process group, and the terminal is
  disassociated from the session
- Any POSIX named semaphores that are open in the calling process are closed as though _sem\_close()
  were called
- Any POSIX message queues that are open in the calling process are closed as though _mq\_close()_
  were called
- If, as a consequence of this process exiting, a process group becomes orphaned and there are any
  stopped processes in that group, then all processes in the group are sent a _SIGHUP_ signal
  followed by a _SIGCONT_ signal
- Any memory locks established by this proces using _mlock()_ or _mlockall()_ are removed
- Any memory mappings established by this process using _mmap()_ are unmapped

### Exit Handlers

Sometimes, an application needs to automatically perform some operaetions on process termination.
One approach in such situations is to use an _exit handler_. An exit handler is a programmer-supplied
function that is registered at some point during the life of the process and is then automatically
called during _normal_ process termination via _exit()_. Exit handlers are not called if a program
calls _\_exit()_

Some notes about registering exit handlers:

- _atexit()_ can be used to register exit handlers. However, it suffers a couple of limitations. The
  first is that when called, an exit handler doesn't know what status was passed to _exit()_. Knowing
  the status can be useful in case an user want to perform different actions depending on whether
  the process is exiting successfully or unsuccessfully. The second limitation is that we can't specify
  an argument to the exit handler when it is called. Such a facility could be useful to define an exit
  handler that performs different actions depending on its argument, or to register a function multiple
  times, each time with a different argument
- _on\_exit()_ mitigates the above problem
- A child process created via _fork()_ inherits a copy of its parent's exit handler registrations. When
  a process performs an _exec()_, all exit handler registrations are removed.
- Multiple exit handlers can be registered. The invocation order is in the reverse

Some notes about _fork()_ interaction with _exit()_ and _\_exit()_

- _exit()_ flush the stdio buffer
- _\_exit()_ does not flush the stdio buffer
- If a child process created by fork(), it has the copy of of the user _stdio_ buffer, so we should
  either use _fflush()_ to flush the _stdio_ buffer prio to a _fork()_ call (we can also disable
  buffering) or calls _exit()_ for parent process but _\_exit()_ for child processes