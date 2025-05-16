---
title: "Linux Series: Daemons"
date: 2025-05-14
---
A _daemon_ is a process with the following characteristics:

- It is long-lived. Often, a daemon is created at system startup and runs until the system is shut down
- It runs in the background and has no controlling terminal. The lack of a controlling terminal ensures
  that the kernel never automatically generates any job-control or terminal-related signals (such as
  _SIGINT_, _SIGTSTP_, and _SIGHUP_) for a daemon

## Creating a Daemon

To become a daemon, a program performs the following steps:

1. Perform a _fork()_, after which the parent exits and the child continues. This step is done for two
   reasons:
   - Assuming the daemon was started from the command line, the parent's termination is noticed by the
     shell, which then displays another shell prompt and leaves the child to continue in the background
   - The child process is guaranteed not to be a process group leader, since it inherited its process
     group ID from its parent and obtained its own unique process ID, which differs from the inherited
     process group ID. this is required in order to be able to successfully perform the next step
2. The child process calls _setsid()_ to start a new sessions and free itself of any association with
   a controlling terminal
3. If the daemon never opens any terminal devices thereafter, then we don't need to worry about the
   daemon reacquiring a controlling terminal. If the daemon might later open a terminal device, then
   we must take steps to ensure that the device does not become the controlling terminal. We can do 
   this in two ways:
   - Specify the _O\_NOCTTY_ flag on any _open()_ that may apply to a terminal device
   - Alternatively, and more simply, perform a second _fork()_ after the _setsid()_ call, and again
     have the parent exit and the (grand)child continue. This ensures that the child is not the
     session leader, and thus, the process can never reacquire a controlling terminal
4. Clear the process umask, to ensure that, when the daemon creates files and directories, they have
   the requested permissions
5. Change the process's current working directory, typically to the root directory (/). This is
   necessary because a daemon usually runs until system shutdown; if the daemon's current working
   directory is on a file system other the one containing /, then that file system can't be unmounted.
   Alternatively, the deamon can change its working directory to a location where it does its job or
   a location defined in its configuration file, as long as we know that the file system containing
   this directory never needs to be unmounted
6. Close all open file descriptors that the daemon has inherited from its parent. Since the daemon has
   lost its controlling terminal and is running in the background, it makes no sense for the daemon to
   keep file descriptors 0, 1, and 2 open if these refer to the terminal. Furthermore, we can't unmount
   any file system on which the long-lived daemon holds files open. And, as usual, we should close unused
   open file descriptors because file descriptors are a finite resource
7. After having closed file descriptors 0, 1, and 2, a daemon normally opens _\/dev\/null_ and uses
   _dup2()_ to make all those descriptors refer to this device. This is done for two reasons:
   - It ensures that if a daemon calls library functions that perform I/O on these descriptors, those
     functions won't unexpectedly fail
   - It prevents the possibility that the daemon later opens a file using descriptor 1 or 2, which is
     then written to-and thus corrupted-by a library function that expects to treat these descriptors
     as standard output and standard error