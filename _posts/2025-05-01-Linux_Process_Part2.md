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