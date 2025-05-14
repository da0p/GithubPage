---
title: "Linux Series: Process Groups, Sessions, and Job Control"
date: 2025-05-11
---
Process groups and sessions form two-level hierarchical relationship between processes:

- A process group is a collection of related processes
- A session is a collection of related process groups

Process groups and sessions are abstractions defined to support shell job control, which allows
interactive users to run commands in the foreground or in the background. The term _job_ is often
used synonymously with the term _process group_

![Process Group](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/process_group.drawio.png)

A process group has a _lifetime_, which is the period of time beginning when the leader creates the
group and ending when the last member process leaves the group. The process group leader need not
be the last member of a process group.

A _session_ is a collection of a process groups. A process's session membership is determined by its
_session identifier_(SID), which, like the process group ID, is a number of type _pid\_t_. A _session leader_
is the process that creates a new session and whose process ID becomes the session ID. 

All of the processes in a session share a single _controlling terminal_. The controlling terminal
is established when the session leader first opens a terminal device. A terminal may be the controlling
terminal of at most one session.

At any point in time, one of the process groups in a session is the _foreground process group_ for
the terminal, and the others are _background process groups_. Only processes in the foreground
process group can read input from the controlling terminal. When the user types one of the signal-generating
terminal characters on the controlling terminal, a signal is sent to all members of the foreground
process group

The main use of sessions and process groups is for shell job control. Each command or pipeline of
commands started from the shell results in the creation of one or more processes, and the shell
places all of these processes in a new process group.

## Controlling Terminals and Controlling Processes

All of the processes in a session may have a (single) controlling terminal. Upon creation, a session
has no controlling terminal; the controlling terminal is established when the session leader first
opens a terminal that is not already the controlling terminal for a session, unless the _O\_NOCTTY_
flag is specified when calling _open()_. A terminal may be the controlling terminal for at most one
session

The controlling terminal is inherited by the child of a _fork()_ and preserved across an _exec()_.
When a session leader opens a controlling terminal, it simultaneously becomes the controlling process
for the terminal. If a terminal disconnect subsequently occurs, the kernel sends the controlling process
a _SIGHUB_ signal to inform it of this event.

If a process has a controlling terminal, opening the special file _\/dev\/tty_ obtains a file descriptor
for that terminal. If the process doesn't have a controlling terminal, opening _\/dev\/tty_ fails with
the error _ENXIO_

## Foreground and Background Process Groups

The controlling terminal maintains the notion of a foreground process group. Wihtin a session, only
one process can be in the foreground at a particular moment; all of the other process groups in the
session are background process groups. The foreground process group is the only process group that can
freely read and write on the controlling terminal. When of the signal-generating germinal characters
is typed on the controlling terminal, the terminal drive delivers the corresponding signal to the
members of the foreground process group.

## The _SIGHUP_ Signal

When a controlling process loses its terminal connection, the kernel sends it a _SIGHUP_ signal to
inform it of this fact. Typically, this may occur in two circumstances:

- When a "disconnect" is detected by the terminal driver, indicating a loss of signal on a modem or
  terminal line
- When a terminal window is closed on a workstation. This occurs because the last open file descriptor
  for the master side of the pseudoterminal associated with the terminal window is closed

The delivery of _SIGHUP_ to the controlling process can set off a kind of chain reaction, resulting
in the delivery of _SIGUP_ to many other processes. This may occur in two ways:

- The controlling process is typically a shell. The shell establishes a handler for _SIGHUP_, so that,
  before terminating, it can send a _SIGHUP_ to each of the jobs that it has created. This signal
  terminates those jobs by default, but if instead they catch the signal, then they are thus informed
  of the shell's demise
- Upon termination of the controlling process for a terminal, the kernel disassociates all processes
  in the session from the controlling terminal, disassociates the controlling terminal from the
  session (so that it may be acquired as the controlling terminal by abother session leader), and
  informs the members of the foreground process group of the terminal of the loss of their controlling
  terminal by sending them a _SIGHUP_ signal

### Handling of _SIGHUP_ by the Shell

Most shells are programmed so that, when run interactively, they establish a handler for _SIGHUP_.
This handler terminates the shell, but beforehand sends a _SIGHUP_ signal to each of the process
groups (both foreground and background) created by the shell

### _SIGHUP_ and Termination of the Controlling Process

If the _SIGHUP_ signal that is sent to the controlling process as the result of a terminal disconnect
causes the controlling process to terminate, then _SIGHUP_ is sent to all of the members of the
terminal's foreground process group. This behavior is a consequence of the termination of the controlling
process, rather than a behavior associated specifically with the _SIGHUP_ signal. If the controlling
process terminates for any reason, then the foreground process group is signaled with _SIGHUP_

## Job Control

Job control permits a shell user to simultaneously execute multiple commands (jobs), one in the
foreground and the others in the background. Jobs can be stopped and resumed, and moved between the
foreground and background

![Job Control](https://raw.githubusercontent.com/da0p/GithubPage/main/docs/assets/job_control.drawio.png)

### Handling Job-Control Signals

Since operation of job control is transparent to most applications, they don't need to take special
action for dealing with job-control signals. However, screen-handling programs need to handle the
terminal stop signal (_SIGTSTP_)

### Orphaned Process Groups (and _SIGHUP_ Revisited)

A process group is an orphaned progress group if the parent of every member is either itself a member
of the group or is not a member of the group's session. Put another way, a process group is not
orphaned if at least one of its members has a parent in the same session but in a different process
group.

If there is a stopped process in an orphaned process group, then all members of the group are sent a
_SIGHUP_ signal, to inform them that they have become disconnected from their session, followed by a
_SIGCONT_ signal, to ensure that they resume execution. If the orphaned process group doesn't have
any stopped members, no signals are sent.

A process group may become orphaned either because the last parent in a different process group in
the same session terminated or because of the termination of the last process within the group that
had a parent in another group
